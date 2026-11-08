# Blender 3D Plugin for Antigravity

Integrates the official **Blender Lab Model Context Protocol (MCP) Server** ([blender.org/lab/mcp-server](https://www.blender.org/lab/mcp-server/)) into Google Antigravity, enabling natural-language 3D scene inspection, procedural `bpy`/`bmesh` modeling, shader graph authoring, viewport capture, Cycles/EEVEE rendering, and instant lookup across Blender's Python API and User Manual.

## Features

- **Official Blender Lab MCP Bridge**: Connects Antigravity to a live Blender 5.1+ session over a non-blocking local TCP socket (`localhost:9876`) or runs headless background tasks via `blender --background`.
- **26 Specialized MCP Tools**: Full coverage of code execution (`execute_blender_code`), progressive scene/data-block inspection (`get_objects_summary`, `get_object_detail_summary`), viewport screenshots, rendering, workspace navigation, and offline documentation search (`search_api_docs`, `search_manual_docs`).
- **Procedural 3D Modeling & Faceted Character Engine**: Bundles proven `bmesh` patterns for architectural assets, hard-surface boolean carving, 3D NURBS tubing, and staggered contour lofting for non-voxel, diamond-faceted low-poly characters.
- **Enforced BMesh & Matrix Safety Rules**: Built-in `rules/AGENTS.md` enforcing canonical `T @ R @ S` matrix multiplication order, face normal recalculation, Blender 5.x node tree conventions, and non-destructive workflows.
- **Dedicated Architect Subagent & Modular Skills**: Includes the `blender-3d-architect` subagent alongside two specialized skills (`blender-integration` and `blender-procedural-modeling`).

---

## Prerequisites & Setup

### 1. Install Blender 5.1+ & the Official MCP Add-on
1. Install **[Blender 5.1 or newer](https://www.blender.org/download/)** and ensure the `blender` executable is on your system `PATH` (or set `BLENDER_PATH`).
2. Visit **[blender.org/lab/mcp-server](https://www.blender.org/lab/mcp-server/)** and drag-and-drop the **MCP Add-on** link into an open Blender window (or install from disk via `Edit > Preferences > Get Extensions`).
3. In Blender, go to **`Edit > Preferences > System > Network`** and check **Allow Online Access** $\checkmark$.
4. In **`Edit > Preferences > Add-ons`**, verify **MCP** is enabled and the socket server is running on `localhost:9876` (`Auto Start` is enabled by default).

### 2. Install the `blender-mcp` Server CLI
Install the official `blender-mcp` package onto your `PATH` using `uv` (recommended) or `pipx`:

```bash
uv tool install "git+https://projects.blender.org/lab/blender_mcp.git#subdirectory=mcp"
```

---

## MCP Server Configuration

Configured in `mcp_config.json`:

```json
{
  "mcpServers": {
    "blender": {
      "command": "blender-mcp",
      "args": []
    }
  }
}
```

### Available MCP Tools (26 Tools)

| Category | Tool | Purpose |
| :--- | :--- | :--- |
| **Code Execution** | `execute_blender_code` | Execute Python (`bpy`/`bmesh`) code in the connected interactive Blender session (returns `result` dict). |
| | `execute_blender_code_for_cli` | Execute Python code on a `.blend` file inside a headless `blender --background` process. |
| **Scene & Objects** | `get_objects_summary` | Retrieve the scene's collection hierarchy, parent relationships, and object list. |
| | `get_object_detail_summary` | Inspect an object's transforms, modifiers, materials, and geometry statistics by `name`. |
| **Blend File Summaries** | `get_blendfile_summary_datablocks` | Summarize data-block counts, active workspace, and render engine (`*_for_cli` variant included). |
| | `get_blendfile_summary_path_info` | Check blend file path, save status, age, and backups (`*_for_cli` variant included). |
| | `get_blendfile_summary_missing_files` | Report missing external textures, linked libraries, fonts, or caches (`*_for_cli` variant included). |
| | `get_blendfile_summary_of_linked_libraries` | Inspect direct and indirect linked library trees (`*_for_cli` variant included). |
| | `get_blendfile_summary_usage_guess` | Score (0–100) the primary use-cases of the `.blend` file (`*_for_cli` variant included). |
| **Viewport & Rendering** | `get_screenshot_of_window_as_image` | Capture a PNG screenshot of the entire Blender window. |
| | `get_screenshot_of_area_as_image` | Capture a PNG screenshot of a specific editor area (e.g., `VIEW_3D`). |
| | `get_screenshot_of_window_as_json` | Return a JSON description of the window layout, areas, active object, and selection. |
| | `render_viewport_to_path` | Render the current scene to `output_path` using active render settings. |
| | `render_thumbnail_to_path` | Render a fast, low-resolution thumbnail preview to `output_path`. |
| **UI Navigation** | `jump_to_view3d_object_by_name` | Frame and focus the 3D Viewport on a specific object by `name`. |
| | `jump_to_view3d_object_data_by_name` | Focus the 3D Viewport on the object whose data-block matches `name`. |
| | `jump_to_tab_by_name` | Switch the active workspace tab by `name` (e.g., `"Layout"`, `"Modeling"`, `"Shading"`). |
| | `jump_to_tab_by_space_type` | Switch to a workspace whose primary editor matches `space_type`. |
| **Documentation** | `search_api_docs` | Full-text search across the bundled Blender Python API reference. |
| | `get_python_api_docs` | Retrieve docstrings and signatures for a `bpy` module/class identifier or wildcard (`bpy.ops.mesh.*`). |
| | `search_manual_docs` | Full-text search across the bundled official Blender User Manual. |

---

## Directory Structure

```text
~/.gemini/config/plugins/blender/
├── plugin.json                              # Plugin manifest & starter prompts
├── README.md                                # Documentation, setup, and tool reference
├── mcp_config.json                          # MCP server configuration (blender-mcp)
├── assets/
│   ├── logo.svg                             # Blender vector logo
│   └── blender_icon.png                     # High-res icon asset
├── rules/
│   └── AGENTS.md                            # BMesh, TRS matrix, & non-destructive safety rules
├── agents/
│   └── blender-3d-architect.md              # Specialized 3D Technical Artist & Modeling Architect
└── skills/
    ├── blender-integration/
    │   └── SKILL.md                         # 26 MCP tools workflow & connection guide
    └── blender-procedural-modeling/
        └── SKILL.md                         # Staggered contour lofting, shaders, & studio lighting
```

---

## Activation

Copy `plugins/blender` to `~/.gemini/config/plugins/blender` (or `%USERPROFILE%\.gemini\config\plugins\blender` on Windows) and restart Antigravity.
