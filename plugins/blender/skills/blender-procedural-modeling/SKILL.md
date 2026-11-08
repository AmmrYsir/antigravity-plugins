---
name: blender-procedural-modeling
description: Procedural 3D modeling, hard-surface boolean carving, staggered diamond-faceted organic character generation, shader node graphs, and studio Cycles/EEVEE rendering in Blender 5.x.
---

# Procedural 3D Modeling & Studio Rendering in Blender

Use this skill when generating 3D assets, characters, architectural props, procedural materials, or studio renders in Blender via `bpy` and `bmesh`.

---

## 1. Canonical Matrix Transformations (`T @ R @ S`)

When transforming geometry in `bmesh.ops` (`create_cube`, `create_cone`, `create_icosphere`, `transform`), **always** multiply matrices in `Translation @ Rotation @ Scale` order:

```python
from mathutils import Vector, Matrix, Euler

def trs(loc=(0.0, 0.0, 0.0), rot=(0.0, 0.0, 0.0), scale=(1.0, 1.0, 1.0)) -> Matrix:
    T = Matrix.Translation(Vector(loc))
    R = Euler(rot, 'XYZ').to_matrix().to_4x4()
    S = Matrix.Diagonal(Vector((scale[0], scale[1], scale[2], 1.0)))
    return T @ R @ S
```
> **Warning**: Never compute `Rotation @ Translation`—doing so orbits the translation vector around the world origin `(0, 0, 0)`, displacing parts away from the model.

---

## 2. Organic & Faceted Low-Poly Modeling (Zero Cubes / Non-Voxel)

To construct stylized faceted low-poly characters or organic forms constructed from a **dense network of sharp, angled, non-flat triangles and diamond polygons** (without blocky voxel primitives), use **Staggered Cross-Sectional Contour Lofting**:

1. Define anatomical or organic cross-section rings of $N$ points along the spline/axis.
2. Rotate every odd ring by a half-step ($\Delta\theta = \frac{\pi}{N}$) so vertices stagger between levels.
3. Connect consecutive rings as a triangle strip (`(r1[i], r1[i+1], r2[i])` and `(r1[i+1], r2[i+1], r2[i])`) and enforce flat shading (`poly.use_smooth = False`):

```python
import bpy, bmesh
from mathutils import Vector

def loft_staggered_rings(bm: bmesh.types.BMesh, rings: list[list[Vector]], close_bottom=True, close_top=True):
    N = len(rings[0])
    vert_rings = [[bm.verts.new(co) for co in ring] for ring in rings]

    for r_idx in range(len(vert_rings) - 1):
        r1, r2 = vert_rings[r_idx], vert_rings[r_idx + 1]
        for i in range(N):
            i_next = (i + 1) % N
            bm.faces.new((r1[i], r1[i_next], r2[i]))
            bm.faces.new((r1[i_next], r2[i_next], r2[i]))

    if close_bottom:
        cb = bm.verts.new(sum((v.co for v in vert_rings[0]), Vector()) / N)
        for i in range(N):
            bm.faces.new((vert_rings[0][(i + 1) % N], vert_rings[0][i], cb))

    if close_top:
        ct = bm.verts.new(sum((v.co for v in vert_rings[-1]), Vector()) / N)
        for i in range(N):
            bm.faces.new((vert_rings[-1][i], vert_rings[-1][(i + 1) % N], ct))

    bmesh.ops.recalc_face_normals(bm, faces=bm.faces)
```

---

## 3. Hard-Surface Carving, Wedges & 3D Curves

- **Wedge / Arc Extrusions (Stairs, Gears, Arches)**: Subdivide radial arcs in `bmesh`, construct top/bottom/side faces, call `bmesh.ops.recalc_face_normals`, and attach a non-destructive `BEVEL` modifier (`bevel.width = 0.008`, `bevel.segments = 2`).
- **Continuous 3D Tubing (Railings, Cables, Hair Locks)**: Create a `CURVE` data-block with `dimensions = '3D'`, add a `NURBS` or `BEZIER` spline, and set `bevel_depth` and `bevel_resolution` for smooth tubular geometry.
- **Boolean Carving**: Create a cutter mesh object, add a `BOOLEAN` modifier (`operation = 'DIFFERENCE'`, `solver = 'EXACT'`) on the target object, and hide or apply the cutter.

---

## 4. Blender 5.x Procedural Shader Nodes

In Blender 5.0+, `mat.node_tree` is active by default on new materials (do not set deprecated `mat.use_nodes = True`):

```python
def create_studio_material(name: str, rgba: tuple, roughness=0.48, metallic=0.0):
    mat = bpy.data.materials.new(name=name)
    nodes = mat.node_tree.nodes
    nodes.clear()
    out = nodes.new(type='ShaderNodeOutputMaterial')
    bsdf = nodes.new(type='ShaderNodeBsdfPrincipled')
    bsdf.inputs['Base Color'].default_value = rgba
    bsdf.inputs['Roughness'].default_value = roughness
    bsdf.inputs['Metallic'].default_value = metallic
    mat.node_tree.links.new(bsdf.outputs['BSDF'], out.inputs['Surface'])
    return mat
```

---

## 5. Studio Cyclorama & 4-Point Lighting Setup

For portfolio-grade asset presentation:
1. **Curved Cyclorama Stage**: Extrude a wide floor plane that curves smoothly upward (`+Y` to `+Z`) behind the subject with smooth shading (`use_smooth = True`) and a neutral mid-grey high-roughness material (`Roughness: 0.85`).
2. **4-Point Softbox Rig**:
   - **Key Light (`AREA`)**: Front-right `(2.6, -2.8, 2.3)`, warm tint `(1.0, 0.96, 0.91)`, `size = 2.2`.
   - **Fill Light (`AREA`)**: Front-left `(-2.8, -2.4, 1.8)`, cool tint `(0.82, 0.90, 1.0)`, `size = 3.0`.
   - **Top Softbox (`AREA`)**: Overhead `(0.0, -0.5, 3.2)` to define crown and shoulder planes.
   - **Rim / Kicker (`AREA`)**: Back-left `(-1.2, 2.2, 2.2)` to separate the silhouette from the backdrop.
3. **Camera Lens**: Use a `55mm`–`65mm` portrait focal length (`cam.data.lens = 56.0`) to eliminate wide-angle perspective distortion on full-body character shots.
