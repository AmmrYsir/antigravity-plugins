# MCP Studio Integration Reference

## Overview

Load this reference whenever performing any of the following:

- **MCP operations** — executing Luau code, editing scripts atomically, reading/inspecting DataModel instances, managing assets, or capturing Studio views via the `Roblox_Studio` MCP proxy
- **Autonomous building** — scaffolding game systems, generating 3D geometry/scenes, and iterating safely in Edit mode
- **Debugging via Studio** — reading console output, tracing runtime errors, and applying fixes in-place
- **Project exploration** — mapping an unfamiliar place's instance hierarchy and script layout

---

## MCP Detection & Session Initialization

On every session start or when Studio interaction is requested, verify whether the active **38-tool `Roblox_Studio` MCP proxy** is connected:

### Studio MCP Mode (Official `Roblox_Studio` MCP Proxy — 38 Tools)

**Detect by checking for:** `list_roblox_studios`, `get_studio_state`, `execute_luau`, `multi_edit`, `search_game_tree`, `inspect_instance`, `screen_capture`

**Mandatory Session Handshake (Every Task):**
1. Call `list_roblox_studios()` (`required: []`) to discover open Studio instances (`id`, `name`, `mode`, `state`, `persistedToCloud`, `placeId`, `universeId`, `file`). Confirm with the user if multiple places are open; **`studio_id` is required on all 37 remaining tools**—never hardcode a `studio_id` or spawn `StudioMCP.exe` directly via CLI scripts.
2. Call `get_studio_state(studio_id)` to verify available `datamodel_type` values (`"Edit"`, `"Client"`, `"Server"`). Use `start_stop_play(is_start, studio_id)` to switch between Edit and Play modes cleanly.

### Offline Mode

When no Studio MCP tools are connected, operate in **Offline Mode** — generate copy-paste-ready Luau scripts or Rojo filesystem structures (`default.project.json` + `src/`).

---

## Active `Roblox_Studio` MCP Toolset (38 Tools)

### Session, Mode & Persistence

| Tool | Purpose | Key Rules |
| :--- | :--- | :--- |
| `list_roblox_studios` | List connected Studio instances and retrieve `studio_id` | Call first; match target by `name`, `placeId`, or `file`. |
| `get_studio_state` | Inspect current play state and valid `datamodel_type` values | Call before mode-sensitive operations. |
| `start_stop_play` | Start (`is_start: true`) or stop (`is_start: false`) playtesting | Stop playtesting before Edit-mode script or world edits. |
| `save_place` | Ctrl+S save (cloud version or local `.rbxl`/`.rbxlx`); never publishes | Save only at verified milestones or user request; cloud saves create place versions. |

### Exploration & Script Inspection

| Tool | Purpose | Key Rules |
| :--- | :--- | :--- |
| `search_game_tree` | Flat JSON exploration of DataModel (`datamodel_type`: `Edit`/`Client`/`Server`) | Filter by `path`, `instance_type` (`IsA`), `keywords`; `max_depth` default 3/max 10; `head_limit` default 200. Rejects `"Edit"` during playtest. |
| `inspect_instance` | Detailed properties (`CFrame.Position`, `CFrame.Rotation`), attributes, child counts | Dot-notation `path`; caps multi-match at 20. |
| `script_read` | Read script source with line numbers (`target_file` dot path) | Supports line ranges (`should_read_entire_file: false`). |
| `script_search` | Fuzzy search against script names (`keywords`) | Capped at 10 results; no wildcards. |
| `script_grep` | Search string/Luau pattern across all script sources (`query`) | Capped at 50 matches. |
| `get_console_output` | Read Studio output log (`studio_id`) | Capture runtime errors, warnings, and print traces. |

### Execution, Script Editing & Built-In Helpers

| Tool | Purpose | Key Rules |
| :--- | :--- | :--- |
| `execute_luau` | Execute Luau on Studio main thread (`datamodel_type`: `Edit`/`Client`/`Server`) | Blocks Studio until yield. Split bulk work into chunks (~500–1,000 items), build off-tree, assign properties before `.Parent`, and `task.wait()` once per chunk. |
| `multi_edit` | Atomic multi-edit or creation (`className`) of scripts (`datamodel_type: ["Edit"]`) | Exact `old_string` match (`""` for new script content); respect Rojo ownership when `default.project.json` manages `src/`. |
| `subagent` | Built-in Studio subagent (`explore`, `screen_capture`, `unit_test`) | Autonomous exploration, visual capture, or ModuleScript unit tests. |
| `skill` | Retrieve built-in Roblox Studio skill guides | 13 skills incl. `rbx-docs-search`, `rbx-scene-analysis`, `rbx-import-asset`, `rbx-debug`, `rbx-perf-profiling`, `rbx-unit-test`, `rbx-device-simulator-lua`, `rbx-network-simulator`, `rbx-permissions`, `rbx-open-cloud-usage`. |
| `http_get` | Fetch official `create.roblox.com` `.md`/`llms.txt` or `github.com/Roblox/libmp` | Supports `query` + `context_lines` filtering. |
| `open_cloud_invoke` | Authenticated Roblox Open Cloud API calls & asset uploads | Check scopes with `skill("rbx-permissions")` first. |

### Assets, Generative 3D & Visual Capture

| Tool | Purpose | Key Rules |
| :--- | :--- | :--- |
| `search_asset` | Search Creator Store and user/group/universe inventory | Request explicit user consent before `insert_asset` across different owners. |
| `insert_asset` | Insert asset by numeric `assetId` (`assetName`, `assetType`, `parentPath`) | Always pass `assetName` when known. |
| `generate_procedural_model` | Primitive-part `ProceduralModel` with configurable attributes | Supports `async: true` -> `wait_job_finished`; `partNames` max 8. |
| `generate_mesh` | Textured AI mesh (`maxTriangles` 12–20,000, `size`, `segmentation`) | Supports `async: true` -> `wait_job_finished`. |
| `generate_texture` | Re-texture existing `MeshPart`/`Model` (`Fast` or `Quality`) | Resolves `instance_path` directly without live selection. |
| `generate_material` | Create PBR `MaterialVariant` across 40 `BaseMaterial` enums | Assign returned `BaseMaterial` + `Name` to `BasePart.Material` and `MaterialVariant`. |
| `generate_scene` | Populate models/parts from prompt or images within optional bbox | `useMeshGen` defaults to `true`. |
| `segment_mesh` | Segment a `MeshPart`/`Model` (`parts` max 5) | Supports `async: true` -> `wait_job_finished`. |
| `wait_job_finished` | Poll `jobId` returned by `async: true` generative calls | Default timeout 600s; call only when result is needed. |
| `screen_capture` | Capture Edit-mode viewport image (`capture_id`, optional `camera_position`, `look_at_position`) | Mandatory for visual QA of 3D geometry and UI layouts. |

*(Additional specialized tools: `auto_setup_avatar`, `generate_avatar`, `generate_animation`, `character_navigation`, `user_keyboard_input`, `user_mouse_input`, `store_image`, `upload_image`, `create_skill`, `edit_skill`, `list_skills`, `submit_feedback`.)*

---

## Offline Mode

When no MCP server is connected, generate self-contained Luau code blocks that the user can copy-paste into Studio manually.

### Script Output Format

Always include clear placement instructions:

```luau
-- ==============================================
-- SCRIPT: WeaponSystem
-- PLACE IN: ServerScriptService
-- TYPE: Script (server-side)
-- ==============================================

local WeaponSystem = {}
-- ... implementation ...
return WeaponSystem
```

### For Rojo Users

When the user uses Rojo, Wally, or a filesystem-based workflow, respect `default.project.json` boundaries:

```
src/
├── server/
│   ├── WeaponSystem.server.luau
│   └── DataManager.server.luau
├── client/
│   ├── InputHandler.client.luau
│   └── UIController.client.luau
├── shared/
│   ├── Constants.luau
│   └── Utils.luau
└── default.project.json
```

Include `"$ignoreUnknownInstances": true` on `Workspace` in `default.project.json` whenever Studio-built 3D geometry coexists with Rojo-synced scripts.

### Offline Conventions

- Label every script block with its target service/folder
- Use `ModuleScript`, `Script`, or `LocalScript` type annotations
- Group related scripts together with a section header
- Provide a setup checklist listing manual steps (e.g., creating `RemoteEvent` instances in `ReplicatedStorage`)

---

## Orchestration Patterns

### Autonomous Build

A full build cycle from specification to working game systems or 3D environments.

```
1. DISCOVER & PLAN
   ├── list_roblox_studios + get_studio_state  →  confirm studio_id and "Edit" mode
   ├── search_game_tree(datamodel_type="Edit") + inspect_instance  →  map current state
   └── Check default.project.json for Rojo-owned paths before mutating scripts or Workspace

2. BUILD SYSTEMS & GEOMETRY
   ├── Scripts (non-Rojo): multi_edit(datamodel_type="Edit", className=...)
   ├── Scripts (Rojo-managed): write/edit local src/*.luau files on disk
   └── 3D Geometry: execute_luau(datamodel_type="Edit") wrapped in ChangeHistoryService
       TryBeginRecording/FinishRecording (chunked ~500–1,000 parts off-tree + task.wait())

3. ASSETS & MATERIALS
   ├── search_asset / insert_asset (confirm cross-owner inventory inserts)
   └── generate_material / generate_mesh / generate_procedural_model (async + wait_job_finished)

4. VERIFY (NUMERIC + VISUAL + PLAYTEST)
   ├── 3D Scenes: OBB bounds/clearance/raycast audit + multi-angle screen_capture
   ├── Gameplay: start_stop_play(is_start=true)  →  get_console_output  →  start_stop_play(is_start=false)
   └── Fix issues via multi_edit / execute_luau and re-verify (max 5 iterations)
```

### Debug Loop

Systematic error resolution with bounded retries.

```
1. DETECT
   ├── get_console_output(studio_id)  →  capture errors & stack traces
   └── Parse script path, line number, and failure type

2. LOCATE
   ├── script_grep / script_search  →  find symbol or script path
   ├── script_read(target_file=...)  →  inspect complete surrounding logic
   └── Analyze root cause

3. FIX (IN EDIT MODE)
   ├── If playing, start_stop_play(is_start=false) so Edit-mode changes persist
   └── multi_edit(datamodel_type="Edit") or execute_luau wrapped in TryBeginRecording/FinishRecording

4. VERIFY
   ├── start_stop_play(is_start=true)  →  exercise failing path
   ├── get_console_output(studio_id)  →  confirm zero errors
   └── start_stop_play(is_start=false)

5. ITERATE OR REPORT
   ├── If error persists and attempts < 5  →  return to step 2
   └── If attempts >= 5  →  report findings and remaining unknowns
```

### Bulk Modification

Safe large-scale changes across many instances (`Roblox_Studio` MCP has no native `undo`/`redo` tool, so `ChangeHistoryService` recording is mandatory):

```luau
-- Run via execute_luau(datamodel_type="Edit")
local ChangeHistoryService = game:GetService("ChangeHistoryService")
local recId = ChangeHistoryService:TryBeginRecording("Bulk_Edit", "Bulk Modify Facility Parts")

local ok, err = pcall(function()
    local count = 0
    for _, inst in ipairs(workspace.PosLaju_Facility:GetDescendants()) do
        if inst:IsA("BasePart") then
            inst.Anchored = true
            count += 1
            if count % 500 == 0 then
                task.wait()
            end
        end
    end
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

1. **Search**: `search_game_tree` / `execute_luau` read query to inventory exact target paths and counts.
2. **Execute**: Wrap edits in `TryBeginRecording` / `FinishRecording` (`Commit` on success, `Cancel` on failure), chunking >500-part writes with `task.wait()`.
3. **Verify**: Re-query via `inspect_instance` / `execute_luau` and `screen_capture` to confirm expected state.

### Project Exploration

Understanding an unfamiliar place file quickly:

1. **Hierarchy**: `search_game_tree(datamodel_type="Edit", max_depth=3)` across `Workspace`, `ServerScriptService`, `ReplicatedStorage`, `StarterPlayer`.
2. **Scripts**: `script_search` / `script_grep` for `"require"`, `"RemoteEvent"`, `"BindableEvent"`, then `script_read` on entry points (or delegate to `subagent(subagent_type="explore")` / `skill("rbx-scene-analysis")`).
3. **Architecture**: Map module dependencies, client-server remotes, DataStore flows, and Rojo vs Studio boundaries before proposing edits.

---

## Safety Guidelines

### Pre-Operation Checks

1. **Check Studio ID & State**: Always call `list_roblox_studios` and `get_studio_state` before mutating anything. Do not execute Edit-mode mutations while a playtest is running.
2. **Wrap Mutations in `ChangeHistoryService` Recordings**:
   - Use `TryBeginRecording` -> `pcall` -> `FinishRecording(recId, Enum.FinishRecordingOperation.Commit | Cancel)`.
   - Never pass `nil` to `FinishRecording` (`TryBeginRecording` returns `nil` if another recording is active or during solo playtest), and always pass `Cancel` on error so future recordings are not locked out until Studio restarts.
3. **Read Before Write**: Inspect script contents via `script_read` (or local `read_file`) and DataModel targets via `search_game_tree` / `inspect_instance` before editing.
4. **Verify Instance Existence**: Guard `FindFirstChild` results rather than indexing unverified paths blindly.

### Destructive Operation Safeguards

- **Deleting instances**: Confirm with the user before calling `:Destroy()` on named architectural or gameplay subtrees outside temporary scratch parts.
- **Overwriting scripts**: Read full source first and apply surgical `multi_edit` replacements rather than blind overwrites.
- **Clearing containers**: Never call `:ClearAllChildren()` on top-level services (`Workspace`, `ServerScriptService`, etc.) without explicit user confirmation.

### Playtest Safety

- Mutations applied while `Client`/`Server` playtest datamodels are active are discarded when `start_stop_play(is_start=false)` stops playtesting.
- Always stop playtesting before editing scripts (`multi_edit` requires `datamodel_type="Edit"`) or building persistent world geometry.

---

## Best Practices

### Batch & Off-Tree Construction

Group reads/writes cleanly and build bulk instance trees **off-tree** before parenting:

```luau
local container = Instance.new("Model")
container.Name = "ModularWallChunk"
for i = 1, 200 do
    local part = Instance.new("Part")
    part.Name = "Panel_" .. i
    part.Anchored = true
    part.Size = Vector3.new(4, 12, 1.5)
    part.CFrame = CFrame.new(i * 4, 6, 0)
    part.Parent = container
end
container.Parent = workspace -- single DataModel write after properties are configured
```

### Verify After Modify

Confirm every change took effect:

1. **Scripts**: `script_read` -> `start_stop_play(is_start=true)` -> `get_console_output` -> `start_stop_play(is_start=false)`.
2. **3D Geometry**: `inspect_instance` / OBB-aware `execute_luau` audit -> multi-angle `screen_capture`.
3. **Save Discipline**: Call `save_place(studio_id)` only at verified milestones or when requested by the user.
