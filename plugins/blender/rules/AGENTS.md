# Blender 3D & MCP Agent Guidelines

When operating on Blender scenes, `.blend` files, procedural 3D models, materials, or lighting setups via the `blender` MCP server, adhere strictly to these rules:

## 1. Safety & Non-Destructive Scene Workflows
- **Inspect Before Modifying**: Always query existing collections, objects, materials, and modifiers (`get_objects_summary`, `get_blendfile_summary_datablocks`) before altering an active scene.
- **Structured Return Contract**: Scripts executed via `execute_blender_code` or `execute_blender_code_for_cli` must assign a JSON-serializable `dict` to a top-level variable named `result` (e.g., `result = {"status": "ok", "created": obj.name}`).
- **Data-Block Hygiene**: Creating objects via `bpy.data.objects.new(name, data)` requires linking them explicitly to a target collection (`collection.objects.link(obj)`). When deleting objects, unlink and purge orphaned meshes/materials cleanly.

## 2. BMesh, Transform Matrices & Blender 5.x API Conventions
- **Canonical `T @ R @ S` Matrix Order**: When transforming vertices or primitives in `bmesh.ops`, always compose matrices as `Translation @ Rotation @ Scale`:
  ```python
  T = Matrix.Translation(Vector(loc))
  R = Euler(rot, 'XYZ').to_matrix().to_4x4()
  S = Matrix.Diagonal(Vector((sx, sy, sz, 1.0)))
  mat = T @ R @ S
  ```
  Never multiply `Rotation @ Translation`, which orbits translated coordinates around the world origin `(0, 0, 0)`.
- **Normals & Shading**: After constructing custom `bmesh` faces or lofted loops, always call `bmesh.ops.recalc_face_normals(bm, faces=bm.faces)` before `bm.to_mesh(mesh)` and `bm.free()`.
- **Blender 5.1+ Material Nodes**: Materials and worlds have node trees enabled by default in Blender 5.0+. Access `mat.node_tree.nodes` directly without setting the deprecated `mat.use_nodes = True` property.
- **Dependency Graph Evaluation**: Call `bpy.context.view_layer.update()` or `bpy.context.evaluated_depsgraph_get()` after applying transforms or modifiers before reading world bounding boxes or matrices.

## 3. Procedural 3D Modeling & Aesthetic Quality
- **Avoid Box/Voxel Primitives for Organic Assets**: When asked for stylized, low-poly, or organic characters/creatures, do not stack raw `create_cube` primitives. Construct organic cross-sectional contours (e.g., staggered diamond/triangular ring lofting or subdivided + decimated cages) so triangles follow anatomical and fabric contours naturally.
- **Visual Verification**: Always render a preview frame (`render_viewport_to_path` or `bpy.ops.render.render(write_still=True)`) or capture a viewport screenshot (`get_screenshot_of_window_as_image`) and inspect it to verify camera framing, proportions, normals, and lighting.

## 4. Interactive Socket vs. Background CLI Routing
- **Interactive Mode (`localhost:9876`)**: Use `execute_blender_code`, `get_objects_summary`, and viewport/navigation tools when Blender is open with the MCP extension running.
- **Headless CLI Mode (`blender --background`)**: When Blender is not running interactively or when batch-processing `.blend` files on disk, use the `*_for_cli` tool variants or invoke `blender --background --python <script.py>`.
