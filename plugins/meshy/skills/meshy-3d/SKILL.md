---
name: meshy-3d
description: Generate, texture, rig, animate, remesh, and prepare 3D models for games, 3D printing, and VFX using the Meshy AI MCP server.
---

# Meshy AI 3D Generation & Pipeline Guide

This skill guides you through orchestrating the **Meshy AI MCP Server** (`meshy_*` tools) for text-to-3D, image-to-3D, character rigging, animation, texturing, and 3D printing preparation.

---

## Tool Catalog Overview (28 Tools)

| Category | Available Tools |
| :--- | :--- |
| **3D Generation** | `meshy_text_to_3d`, `meshy_text_to_3d_refine`, `meshy_image_to_3d`, `meshy_multi_image_to_3d` |
| **Creative Lab** | `meshy_creative_lab` (9 product archetypes: figure, lamp, keychain, fridge-magnet, vinyl-figure, brick-figure, keycap, fidget-pixel, fidget-collapsible) |
| **Post-Processing** | `meshy_remesh`, `meshy_retexture` |
| **Rigging & Animation** | `meshy_rig`, `meshy_animate`, `meshy_text_to_motion`, `meshy_list_animations` |
| **Format & Dimensions** | `meshy_convert`, `meshy_resize`, `meshy_uv_unwrap` |
| **2D Image Generation** | `meshy_text_to_image`, `meshy_image_to_image` |
| **Task Management** | `meshy_get_task_status`, `meshy_list_tasks`, `meshy_cancel_task`, `meshy_download_model` |
| **Workspace & Library**| `meshy_list_models` |
| **3D Printing Suite** | `meshy_analyze_printability`, `meshy_repair_printability`, `meshy_process_multicolor`, `meshy_auto_split`, `meshy_send_to_slicer` |
| **Account & Usage** | `meshy_check_balance`, `meshy_list_usage` |

---

## Core Rule: Credit Confirmation

Before calling any tool that costs credits, always state the selected model, state the exact credit cost, and ask the user for confirmation:

| Tool | Cost |
| :--- | :--- |
| `meshy_text_to_3d` (preview mesh) | 20 credits (`meshy-7.1` / `latest`), 5 credits (`meshy-6-lite`, `meshy-t2`, `meshy-t2.5`) |
| `meshy_image_to_3d` (mesh only) | 20 credits (`meshy-7.1` / `latest`), 5 credits (`meshy-6-lite`, `meshy-t2`, `meshy-t2.5`) |
| `meshy_image_to_3d` (+ 2K/4K texture) | 30 credits (`meshy-7.1`), 15 credits (`meshy-6-lite`, `meshy-t2`, `meshy-t2.5`) |
| `meshy_image_to_3d` (+ 8K texture) | 35 credits (`meshy-7.1`), 20 credits (`meshy-t2`, `meshy-t2.5`) |
| `meshy_text_to_3d_refine` / `meshy_retexture` | 10 credits (15 credits for 8K texture resolution); `meshy-n1` is 10 credits |
| `meshy_remesh` / `meshy_rig` / `meshy_uv_unwrap` | 5 credits each (Rigging includes walk + run clips free) |
| `meshy_animate` | 3 credits per action clip |
| `meshy_text_to_motion` | 10 credits (`prime` FBX) / 3 credits (`swift` BVH) |
| `meshy_repair_printability` / `meshy_process_multicolor` / `meshy_auto_split` | 10 credits each |
| `meshy_convert` / `meshy_resize` | 1 credit each |
| `meshy_creative_lab` | 36 credits (keycap: 62 credits, fidget-collapsible: 6 credits) |
| `meshy_text_to_image` | 3–9 credits (`nano-banana`), 9–12 credits (`gpt-image`) |
| **Free queries** | `meshy_check_balance`, `meshy_get_task_status`, `meshy_list_tasks`, `meshy_list_models`, `meshy_download_model`, `meshy_analyze_printability`, `meshy_list_animations`, `meshy_send_to_slicer` |

---

## Workflow Scenarios

### Scenario 1: Game Engine Assets (Unity / Unreal / Godot)
1. **Model & Topology Choice**:
   - Game characters or props need low/medium clean polycounts.
   - Use `model_type: "smart-topology"` with `ai_model: "meshy-t2.5"` (or `"meshy-t2"`) and `target_polycount: 5000` to `15000`.
   - Set `target_formats: ["fbx", "glb"]`.
2. **Texture Choice**:
   - Pair with `texture_model: "meshy-n1"` (2K reference texture) or `"meshy-7"` (PBR texture maps).
3. **Download**:
   - Run `meshy_get_task_status` until `SUCCEEDED`.
   - Call `meshy_download_model` with `task_id` and `format: "fbx"`.

### Scenario 2: Character Rigging & Animation
1. **Generation in T-Pose**:
   - Call `meshy_image_to_3d` or `meshy_text_to_3d` with `pose_mode: "t-pose"`.
   - Ensure the model is a humanoid character under 300,000 faces.
2. **Auto-Rigging**:
   - Call `meshy_rig` with the completed task ID (5 credits).
   - This automatically embeds a standard biped skeleton and skin weights, and bundles walking and running cycles for free!
3. **Additional Animations**:
   - Run `meshy_list_animations` (free) to find preset IDs (e.g., jump, attack, idle).
   - Call `meshy_animate` with `action_ids` (up to 10 clips packed in one FBX, 3 credits per clip).
   - Alternatively, generate custom motion clips with `meshy_text_to_motion`.

### Scenario 3: Single-Color 3D Printing (FDM / Resin)
1. **Geometry Generation**:
   - Generate untextured model (`should_texture: false`, `target_formats: ["obj", "stl"]`).
2. **Printability Audit**:
   - Call `meshy_analyze_printability` (free) with the model or task ID.
   - Inspect report: watertightness, non-manifold edges, volume, thin walls.
3. **Topology Repair**:
   - If defects are found, run `meshy_repair_printability` (10 credits).
4. **Export & Slicer Launch**:
   - Call `meshy_download_model` with `print_ready: true` and desired `print_height_mm`.
   - Run `meshy_send_to_slicer` with `slicer_type: "auto"` to detect installed slicers (OrcaSlicer, Bambu Studio, PrusaSlicer, Cura, Creality, Elegoo, Anycubic) and output launch commands.

### Scenario 4: Multicolor 3D Printing (Bambu Lab AMS / Prusa MMU)
1. **Generate Textured Model**:
   - Run `meshy_image_to_3d` or `meshy_text_to_3d_refine` to get high quality color textures.
2. **Process Multicolor Segmentation**:
   - Call `meshy_process_multicolor` with `input_task_id`, `style: "realistic"` or `"cartoon"`, and the target printer/slicer preset (10 credits).
3. **Download 3MF**:
   - Download the model formatted as `3mf`.
4. **Open in Slicer**:
   - Call `meshy_send_to_slicer` with `is_multicolor: true` to launch the appropriate multi-color slicer.

### Scenario 5: Large Model Auto-Split
1. **Pre-flight**:
   - Resize model to real-world dimensions using `meshy_resize` if necessary.
2. **Partitioning**:
   - Call `meshy_auto_split` with `mode: "auto"`, `"by_parts"`, or `"by_color"`.
   - Specify `connectors: true` for male/female alignment pins and `layout: "on_plate"` for direct bed printing.
3. **Export**:
   - Download the split assembly in `3mf` format with separated printable bodies.

---

## Slicer Detection Reference
The `meshy_send_to_slicer` tool automatically detects common Windows and macOS slicer installations in standard locations:
- **Bambu Studio**: `C:\Program Files\Bambu Studio\bambu-studio.exe`
- **OrcaSlicer**: `C:\Program Files\OrcaSlicer\orca-slicer.exe`
- **PrusaSlicer**: `C:\Program Files\Prusa3D\PrusaSlicer\prusa-slicer.exe`
- **Creality Print**: `C:\Program Files\Creality\Creality Print\CrealityPrint.exe`
- **Elegoo Cura / Ultimaker Cura**: `C:\Program Files\Ultimaker Cura *\Ultimaker-Cura.exe`
- **Anycubic Photon Workshop**: standard Program Files path
