---
name: roblox-studio-building
description: >-
  Procedural 3D building, architectural math, ChangeHistoryService undo safety,
  Rojo boundary discipline, and automated OBB + multi-angle screen_capture QA
  for Roblox Studio Edit mode via the official Roblox_Studio MCP server.
  Use when constructing or auditing 3D buildings, environments, props,
  MaterialVariants, or spatial layouts in Studio.
---

# Roblox Studio 3D Building & Spatial QA Skill

Use this skill alongside the `@roblox-studio-builder` agent (or directly when building/verifying 3D environments in Roblox Studio) for exact geometry math, Studio MCP execution discipline, Rojo boundaries, and automated verification snippets.

## Reference Routing

| Topic | Reference File | Load When |
| :--- | :--- | :--- |
| **Scale, Geometry, Lighting & Materials** | [geometry-and-scale.md](./references/geometry-and-scale.md) | Checking `WedgePart`/`CornerWedgePart`/`TrussPart` axes, `CFrame` math, R6/R15 stud proportions, Z-fighting/voxel light-leak thresholds, CSG limits, or `MaterialVariant` vs `SurfaceAppearance`. |
| **MCP Workflow, Undo, Rojo & Validation** | [studio-mcp-and-validation.md](./references/studio-mcp-and-validation.md) | Running `Roblox_Studio` MCP Edit-mode builds, `ChangeHistoryService` recordings, Rojo `$ignoreUnknownInstances` checks, OBB-aware Luau audits, raycast probes, or multi-angle `screen_capture` QA. |

## Core Execution Workflow

1. **Discover & Lock Edit Mode**:
   - `list_roblox_studios()` -> select `studio_id` -> `get_studio_state(studio_id)`.
   - Confirm `"Edit"` datamodel is active (`start_stop_play(is_start=false)` if stuck in Play mode, with user confirmation when appropriate).
   - Inspect `default.project.json` (Rojo boundaries) and live Studio hierarchy (`search_game_tree` / `inspect_instance`).
2. **Define Coordinate & Naming Contract**:
   - Fix floor `Y` datum (e.g., `Y = 0`), world origin, exterior/interior zone bounds, doorway apertures, and exact bay centerlines before building.
3. **Build Safely in Chunks**:
   - Wrap all Edit-mode mutations in `ChangeHistoryService:TryBeginRecording` / `FinishRecording(..., Commit | Cancel)`.
   - Construct ~500–1,000 parts off-tree before parenting each chunk and calling `task.wait()`.
   - Use `generate_procedural_model`, `generate_mesh`, `generate_material`, `generate_texture`, `generate_scene`, or `search_asset`/`insert_asset` when native meshes/materials beat primitive blockouts.
4. **Verify Numerically & Visually**:
   - Run the OBB-aware Luau audit (`Anchored`, `Size`, Z-fighting pairs, `GetPartBoundsInBox` doorway clearance, interior sky-leak raycasts, cross-zone parity).
   - Capture and visually inspect multi-angle `screen_capture` views (isometric, all 4 elevations, interior horizon toward doors/eaves, ground-level curb close-ups).
5. **Keep Repo Clean**:
   - Never write ad-hoc `.luau`, `.py`, or `.png` files into the project root; pass Luau inline or store temporary dumps in session `scratch/`.
