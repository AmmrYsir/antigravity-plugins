# Meshy AI 3D Generation & MCP Rules

When interacting with the Meshy MCP server (`meshy_*` tools) to generate, optimize, rig, or print 3D assets, follow these mandatory rules:

## 1. Mandatory Credit Cost Confirmation
- **Ask Before Calling Any Paid Tool**: Meshy tasks cost credits on the user's account. Never trigger a paid generation, refine, remesh, rig, or printing repair task without stating the model choice, the exact credit cost, and asking the user for confirmation.
- **Credit Reference**:
  - `meshy_text_to_3d` (preview): 20 credits (`meshy-7.1` / `latest` / `meshy-6`), 5 credits (`meshy-6-lite`, `meshy-t2`, `meshy-t2.5` Smart Topology).
  - `meshy_image_to_3d` / `meshy_multi_image_to_3d`:
    - Geometry: `meshy-7.1` / `latest` (20 credits mesh, 30 textured 2K/4K, 35 textured 8K).
    - Lightweight: `meshy-6-lite` (5 credits mesh, 15 textured 2K).
    - Smart Topology: `meshy-t2` / `meshy-t2.5` (5 credits mesh, 15 textured 2K, 20 textured 8K).
  - `meshy_text_to_3d_refine` / `meshy_retexture`: 10 credits (15 credits for 8K texture resolution); `meshy-n1` retexture is 10 credits (2K base color only).
  - `meshy_remesh` / `meshy_rig` (includes walk + run animations) / `meshy_uv_unwrap`: 5 credits each.
  - `meshy_animate`: 3 credits per action clip.
  - `meshy_text_to_motion`: 10 credits (`prime` FBX) / 3 credits (`swift` BVH).
  - `meshy_repair_printability` / `meshy_process_multicolor` / `meshy_auto_split`: 10 credits each.
  - `meshy_convert` / `meshy_resize`: 1 credit each.
  - `meshy_creative_lab`: 36 credits (keycap: 62 credits, fidget-collapsible: 6 credits).
  - `meshy_text_to_image` / `meshy_image_to_image`: 3–9 credits (`nano-banana`), 9–12 credits (`gpt-image`).
  - **Free Operations**: `meshy_check_balance`, `meshy_list_usage`, `meshy_list_tasks`, `meshy_list_models`, `meshy_get_task_status`, `meshy_download_model`, `meshy_analyze_printability`, `meshy_send_to_slicer`, and `meshy_list_animations`.

## 2. Model Selection Discipline by Project Goal
- **Game Engine & Real-Time Assets (Unity, Unreal, Godot)**:
  - Default to **Smart Topology** (`model_type: "smart-topology"`) with `ai_model: "meshy-t2.5"` or `"meshy-t2"` and an explicit `target_polycount` (e.g. 5,000–15,000 triangles).
  - Pair with texture model `meshy-n1` or `meshy-7`.
  - Avoid creating dense `meshy-7.1` sculpts and attempting to decimate/remesh them down afterwards.
- **Cinematic & Hero Props**:
  - Use `meshy-7.1` + `meshy-7` (PBR textures up to 8K).
- **Humanoid Character Animation**:
  - Always generate in `pose_mode: "t-pose"`.
  - Rigging (`meshy_rig`) requires a textured humanoid under 300K faces.
  - Rigging includes walking and running animations for free. For additional animations, check `meshy_list_animations` first to obtain valid action IDs.
- **3D Printing (FDM / SLA)**:
  - Single-color: Generate mesh-only (`target_formats: ["obj"]` or `["stl"]`, `should_texture: false`), run `meshy_analyze_printability` (free), repair if needed, and scale with `print_height_mm`.
  - Multicolor (Bambu AMS / Prusa MMU): Generate textured model, run `meshy_process_multicolor`, export as `3mf`, and launch via `meshy_send_to_slicer`.
  - Large models: Use `meshy_auto_split` with connectors and `layout: "on_plate"`.

## 3. Safe Task & Polling Operations
- **Never Re-create an Ambiguous Task**: If a task creation call times out or encounters a network error, check `meshy_list_tasks` before repeating the request to prevent double-billing.
- **Respect Polling Intervals**: When waiting for long-running generation, query `meshy_get_task_status` every 5 to 15 seconds until the status is `SUCCEEDED` or `FAILED`.
- **Target Formats**: Specify desired formats (`glb`, `fbx`, `obj`, `usdz`, `stl`, `3mf`) upfront in `target_formats` during task creation whenever possible.
