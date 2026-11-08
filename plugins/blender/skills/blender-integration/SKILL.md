---
name: blender-integration
description: Interact with Blender 3D (5.1+) via the official Blender Lab MCP server to inspect scenes, execute Python/BMesh code, capture viewports, render previews, and search bundled Blender Python API and manual documentation.
---

# Blender 3D MCP Integration Guide

This skill provides the workflow and tool reference for controlling and inspecting **Blender 3D** using the official **Blender Lab MCP Server** (`blender-mcp`).

## Architecture & Connection Modes

The official Blender MCP integration consists of two components:
1. **Blender Add-on (`mcp`)**: Runs inside Blender (5.1+) and opens a non-blocking local TCP socket server on `localhost:9876` (configurable via `BLENDER_MCP_HOST` and `BLENDER_MCP_PORT`).
2. **MCP Server (`blender-mcp`)**: Launched by Antigravity over `stdio` to bridge MCP tool calls to Blender's socket server or headless CLI (`BLENDER_PATH`).

```text
Antigravity ⇐ MCP / stdio ⇒ blender-mcp ⇐ TCP localhost:9876 ⇒ Blender Add-on
```

### Mode A: Interactive GUI Session
1. Open **Blender 5.1+**.
2. Enable **Allow Online Access** in `Edit > Preferences > System > Network`.
3. Enable the **MCP** extension in `Edit > Preferences > Add-ons` and ensure the server is started (`Auto Start` is enabled by default on port `9876`).

### Mode B: Headless / Background CLI Mode
- Use the `*_for_cli` tools (which automatically invoke `blender --background <blend_file>`) or run a standalone generator script via:
  ```bash
  blender --background --python generate_scene.py
  ```
- You can also run the MCP socket bridge in a headless terminal session:
  ```bash
  blender --online-mode --background scene.blend --command blender_mcp
  ```

---

## Complete Catalog of 26 MCP Tools

### 1. Code Execution
| Tool | Description |
| :--- | :--- |
| `execute_blender_code` | Execute Python (`bpy` / `bmesh`) code in the connected interactive Blender instance. Must assign a JSON-serializable dict to `result = {...}`. |
| `execute_blender_code_for_cli` | Execute Python code against a target `blend_file` in a background Blender process (`blender --background`). |

### 2. Scene & Data-Block Inspection
| Tool | Description |
| :--- | :--- |
| `get_objects_summary` | Return the active scene's collection hierarchy, parent relationships, and objects. |
| `get_object_detail_summary` | Return a structured summary of a specific object by `name` (transforms, modifiers, materials, mesh stats). |
| `get_blendfile_summary_datablocks` | Return data-block counts, active workspace, and render engine (`*_for_cli` variant available). |
| `get_blendfile_summary_path_info` | Return blend file path, save status, age, and backup files (`*_for_cli` variant available). |
| `get_blendfile_summary_missing_files` | Report missing external file references such as textures, libraries, fonts, or caches (`*_for_cli` variant available). |
| `get_blendfile_summary_of_linked_libraries` | Return a hierarchy tree of directly and indirectly linked `.blend` library files (`*_for_cli` variant available). |
| `get_blendfile_summary_usage_guess` | Score (0–100) the primary use-cases of the current `.blend` file (`*_for_cli` variant available). |

### 3. Viewport Capture & Rendering
| Tool | Description |
| :--- | :--- |
| `get_screenshot_of_window_as_image` | Capture a PNG screenshot of the entire Blender application window. |
| `get_screenshot_of_area_as_image` | Capture a PNG screenshot of a single Blender editor area (e.g., `VIEW_3D`). |
| `get_screenshot_of_window_as_json` | Return a JSON layout description of the Blender window, areas, active object, and selection. |
| `render_viewport_to_path` | Render the current scene to `output_path` using active render settings. |
| `render_thumbnail_to_path` | Render a fast, low-resolution thumbnail preview to `output_path`. |

### 4. Interface & Viewport Navigation
| Tool | Description |
| :--- | :--- |
| `jump_to_view3d_object_by_name` | Frame and focus the 3D Viewport camera/view on an object by `name`. |
| `jump_to_view3d_object_data_by_name` | Focus the 3D Viewport on the object whose underlying data-block matches `name`. |
| `jump_to_tab_by_name` | Switch the active Blender workspace tab by `name` (e.g., `"Layout"`, `"Modeling"`, `"Shading"`). |
| `jump_to_tab_by_space_type` | Switch to a workspace whose main editor matches `space_type`. |

### 5. Bundled Documentation Search
| Tool | Description |
| :--- | :--- |
| `search_api_docs` | Full-text search across the bundled Blender Python API reference (`.rst` docs). |
| `get_python_api_docs` | Retrieve detailed documentation for a `bpy` module/class identifier or wildcard pattern (`bpy.ops.mesh.*`). |
| `search_manual_docs` | Full-text search across the bundled official Blender User Manual. |

---

## Execution Best Practices

1. **Always Assign `result`**:
   ```python
   import bpy
   result = {
       "version": bpy.app.version_string,
       "objects": [obj.name for obj in bpy.context.scene.objects],
   }
   ```
2. **Progressive Scene Exploration**: On large production files, call `get_blendfile_summary_datablocks` and `get_objects_summary` first, then drill into individual objects with `get_object_detail_summary`.
3. **Evaluate Dependency Graph**: When calculating polygon counts or world-space coordinates after modifiers, evaluate via `depsgraph = bpy.context.evaluated_depsgraph_get()`.
