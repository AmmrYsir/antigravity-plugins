# Roblox Studio MCP Workflow, Undo Safety, Rojo & Validation Reference

## 1. Active `Roblox_Studio` MCP Toolset (38 Tools)
Do **not** assume legacy community (`get_file_tree`, `get_instance_properties`, `undo`, `redo`, `create_build`) or archived `studio-rust-mcp-server` (`run_code`, `get_studio_mode`, `export_place_to_rbxlx`) tool names. Use the active `Roblox_Studio` MCP proxy tools:

| Category | Tools | Key Parameters & Guardrails |
| :--- | :--- | :--- |
| **Session & State** | `list_roblox_studios`, `get_studio_state`, `start_stop_play`, `save_place` | Call `list_roblox_studios()` first (`studio_id` required on 37/38 tools). Verify `"Edit"` datamodel via `get_studio_state` before mutating. Use `save_place` only at major milestones (never after every edit—cloud saves create versions). |
| **Explore & Read** | `search_game_tree`, `inspect_instance`, `script_read`, `script_search`, `script_grep`, `get_console_output` | `search_game_tree` rejects `datamodel_type="Edit"` during playtest (`max_depth` default 3/max 10, `head_limit` default 200). `inspect_instance` caps multi-match at 20. |
| **Edit & Execute** | `execute_luau`, `multi_edit` | `execute_luau` blocks Studio main thread until yield: build ~500–1,000 parts off-tree before parenting and call `task.wait()` once per chunk. `multi_edit` requires `datamodel_type="Edit"` and `className` for new scripts. |
| **Assets & GenAI** | `search_asset`, `insert_asset`, `generate_procedural_model`, `generate_mesh`, `generate_texture`, `generate_material`, `generate_scene`, `segment_mesh`, `wait_job_finished` | Generative tools support `async: true` -> poll with `wait_job_finished`. `search_asset` cross-owner inventory inserts require explicit user consent. `generate_material` returns `BaseMaterial` + `MaterialVariant` Name. |
| **Visual QA & Built-ins** | `screen_capture`, `subagent`, `skill`, `http_get`, `open_cloud_invoke` | `screen_capture(capture_id, studio_id, camera_position, look_at_position)` captures Edit-mode viewport. Built-in MCP skills include `rbx-docs-search`, `rbx-scene-analysis`, `rbx-import-asset`, `rbx-permissions`. |

---

## 2. `ChangeHistoryService` Edit-Mode Recording Template
Because `Roblox_Studio` MCP has no built-in `undo`/`redo` tool, wrap every mutating `execute_luau` script in an explicit recording:

```luau
local ChangeHistoryService = game:GetService("ChangeHistoryService")
local recId = ChangeHistoryService:TryBeginRecording("Build_Zone", "Build Zone Geometry")
-- TryBeginRecording returns nil if another recording is active or during solo playtest.
-- Never pass nil to FinishRecording, and always Cancel on failure to avoid locking Studio.

local ok, err = pcall(function()
    local container = Instance.new("Model")
    container.Name = "Zone_Chunk"
    -- Create and configure all parts (Anchored = true, Size, CFrame) BEFORE parenting container:
    -- ...
    container.Parent = workspace
end)

if recId then
    ChangeHistoryService:FinishRecording(
        recId,
        if ok then Enum.FinishRecordingOperation.Commit else Enum.FinishRecordingOperation.Cancel
    )
end

if not ok then
    error(err)
end
```

---

## 3. Rojo vs Studio Coexistence Rules
- Always read `default.project.json` before creating or renaming top-level `Workspace` / service containers.
- Inside any `$path`-mapped Rojo tree, `$ignoreUnknownInstances` defaults to `false`—Rojo deletes unmapped Studio instances on sync unless `"$ignoreUnknownInstances": true` is explicitly configured in `default.project.json` or `init.meta.json`.
- Check `Workspace` static definitions in `default.project.json` (e.g., an inherited `Baseplate` with `Position = [0,-10,0]`, `Size = [512,20,512]` whose top surface sits at `Y = 0` and Z-fights with custom ground slabs).
- Binary-backed Studio geometry (`CSG`, `MeshPart.MeshId`, `Terrain`, `MaterialVariant` assets) does not live-sync to `.luau` trees via Rojo. Export snapshots into `.rbxlx` (`rojo build default.project.json --output game.rbxlx --base base.rbxlx`) or `.rbxmx` XML models when versioning Studio-built scenes in Git.

---

## 4. Automated OBB & Geometry Validation Suite (Luau)

### 4.1 OBB-Aware World Bounds (`CFrame:GetComponents()`)
Never compute `p.Position ± p.Size/2` (blind to `CFrame` rotation). Always project the rotated local half-extents:

```luau
local function getPartWorldAABB(p: BasePart): (Vector3, Vector3)
    local x, y, z, r00, r01, r02, r10, r11, r12, r20, r21, r22 = p.CFrame:GetComponents()
    local h = p.Size * 0.5
    local hx = math.abs(r00) * h.X + math.abs(r01) * h.Y + math.abs(r02) * h.Z
    local hy = math.abs(r10) * h.X + math.abs(r11) * h.Y + math.abs(r12) * h.Z
    local hz = math.abs(r20) * h.X + math.abs(r21) * h.Y + math.abs(r22) * h.Z
    local pos = Vector3.new(x, y, z)
    local ext = Vector3.new(hx, hy, hz)
    return pos - ext, pos + ext
end
```

### 4.2 Five Structural Checks Before Visual Capture
Run inline via `execute_luau(datamodel_type="Edit")` against the target root (e.g. `workspace.Facility`):
1. **Anchoring & Size Bounds**: Verify `0` parts have `Anchored == false`; verify every axis is within `[0.001, 2048]` (`[0.05, 2048]` when `CanCollide == true`).
2. **Coplanar / Z-Fighting Pair Scan**: For horizontal slabs (`|UpVector.Y| > 0.999`) or parallel wall faces (`|n1:Dot(n2)| > 0.999`) within the facility OBB, flag pairs whose top/side surface planes are `< 0.005` studs apart while their tangential world footprints overlap by `> 0.05` studs.
3. **Opening Clearance (`GetPartBoundsInBox`)**: For every registered doorway/bay `{ cf, size }`, query `workspace:GetPartBoundsInBox(cf, size - Vector3.new(0.2, 0.2, 0.2), overlapParams)` (filtering out intentional transparent triggers/shutters); fail if any curb, wall, or column intersects the clear passage.
4. **Interior Sky-Leak Raycast Probe**: From interior eye points (`Y = 5`), fire ray fans toward wall-roof eaves, valleys, and bay divider seams (`Enum.RaycastFilterType.Include` on the building shell); fail if any ray returns `nil` (escaping to open sky).
5. **Cross-Zone Alignment Parity**: Compare Exterior vs Interior bay/door world coordinates and assert `math.abs(extX - intX) <= 0.05` and `math.abs(extZ - intZ) <= 0.05`.

---

## 5. Multi-Angle `screen_capture` Verification Matrix

Execute `screen_capture(capture_id, studio_id, camera_position, look_at_position)` and inspect every returned image:

| View ID | Camera Placement | Look Target | Defects Caught |
| :--- | :--- | :--- | :--- |
| `Iso_FrontLeft` / `Iso_RearRight` | 45° elevated outside corner (`Y = height * 1.8`) | Facility center | Overall massing, roof ridge orientation, perimeter fence containment. |
| `Elev_PosX`, `Elev_NegX`, `Elev_PosZ`, `Elev_NegZ` | Straight-on orthogonal outside each facade | Facade center at mid-height | Inverted `WedgePart` gable pediments, valley holes, signage/canopy offsets. |
| `Int_Horizon_Bays` / `Int_Horizon_Doors` | Inside room at avatar eye height (`Y = 5`) | Exterior bay doors / eave line | Sky gaps above walls, vertical light-leak slits between door tracks and posts. |
| `Detail_DoorCurbs` / `Detail_Floor` | Low oblique (`10–15` studs away, `Y = 3..6`) | Door threshold / floor junctions | Continuous curbs clipping doorways, floating doors, coplanar floor Z-fighting. |
