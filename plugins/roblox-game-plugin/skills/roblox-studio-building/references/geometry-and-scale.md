# Roblox Studio 3D Geometry, Scale & Material Reference

## 1. World Scale & Architectural Proportions
- **Engine Unit Conversion**: `1 stud = 0.28 m` (`1 m ≈ 3.571 studs`), `1 RMU = 21.952 kg`, default `Workspace.Gravity = 196.2 studs/s²`.
- **Avatar Dimensions**:
  - Standard R6 / R15 character: ~`5.0–6.5` studs tall × `4` wide × `2` deep (note R15 dynamic scaling can range wider/taller).
- **Architectural Guidelines**:
  - **Personnel Doorways**: minimum `4` studs wide × `7–8` studs tall (avoid `< 3` wide or `< 6` tall).
  - **Stairs**: riser `0.5–1.0` stud (exceeding `1.5` forces jumping), tread `1.5–2.0` studs (`Humanoid.MaxSlopeAngle = 45°` default).
  - **Corridors & Aisles**: minimum `6–8` studs for comfortable two-way player flow; `10–14` studs for industrial forklift/cart aisles.
  - **Vehicles & Loading Docks**:
    - Passenger car: `8–10` studs wide; parking stall `9–10 × 18–20` studs; driving aisle `20–24` studs.
    - Delivery van / box truck: `12–16` studs wide; loading dock bay `14–16` studs wide, dock apron `35–50` studs deep, raised dock platform `2–3` studs if applicable.

---

## 2. `BasePart` Bounds & Primitive Geometry Math

### `BasePart` Limits
- Per-axis `Size` max: **2048 studs**.
- Visual minimum: **0.001 studs**; physics collision minimum: **0.05 studs** (enforce `>= 0.05` on all `CanCollide = true` architectural parts).
- Always set `TopSurface = Enum.SurfaceType.Smooth` and `BottomSurface = Enum.SurfaceType.Smooth` for clean architectural blockouts unless studs/welds are desired.

### `WedgePart` Exact Local Axes (`Orientation = (0, 0, 0)`)
- Local basis: `RightVector = +X`, `UpVector = +Y`, `LookVector = -Z` (`Front`).
- **Flat Base**: `-Y` face (`Y = -Size.Y / 2`).
- **Low Thin Edge**: **`-Z` (`Front` / `LookVector`)**.
- **Tall Vertical Wall**: **`+Z` (`Back`)**.
- **Slope Direction**: Rises from `-Z` (`-Y/2`) up to `+Z` (`+Y/2`).
  - Outward slope normal (local): `Vector3.new(0, Size.Z, -Size.Y).Unit` (faces `-Z` and `+Y`).
  - Pitch angle: `math.atan(Size.Y / Size.Z)` (`CFrame.UpVector` stays `+Y`, not normal to the slope).
  - `CFrame.Angles(0, math.rad(180), 0)` flips tall wall to `-Z` and low thin edge to `+Z`.

**Mandatory Pre-Batch Raycast Verification Snippet** (test 1 wedge before spawning a roof/gable array):
```luau
local w = Instance.new("WedgePart")
w.Size = Vector3.new(2, 6, 10)
w.CFrame = CFrame.new(0, 3, 0)
w.Anchored = true
w.Parent = workspace
local rp = RaycastParams.new()
rp.FilterType = Enum.RaycastFilterType.Include
rp.FilterDescendantsInstances = { w }
local rFront = workspace:Raycast(Vector3.new(0, 10, -4), Vector3.new(0, -20, 0), rp)
local rBack = workspace:Raycast(Vector3.new(0, 10, 4), Vector3.new(0, -20, 0), rp)
w:Destroy()
-- Expected at Orientation=(0,0,0): rFront.Position.Y ≈ 0.60 (low -Z), rBack.Position.Y ≈ 5.40 (tall +Z)
```

### `CornerWedgePart` & `TrussPart`
- **`CornerWedgePart`**: 5 vertices (4 bottom corners + 1 top apex); vertical walls on **`+X`** and **`+Z`**, sloping downward toward **`-X`** and **`-Z`**. Rotate in 90° Y increments for hip-roof outer corners; inside hip valleys require modular wedges or a `MeshPart`.
- **`TrussPart`**: Strict `2 × 2 × n` studs (`n` multiple of 2, max **512**). Does not render `Decal`/`Texture` children.

---

## 3. `CFrame`, Pivots & Axis-Size Parity
- **Euler Rotation Order**:
  - `BasePart.Orientation` (degrees) and `CFrame.fromOrientation(rx, ry, rz)` (= `CFrame.fromEulerAnglesYXZ`) use **YXZ** order.
  - `CFrame.Angles(rx, ry, rz)` (= `CFrame.fromEulerAnglesXYZ`) uses **XYZ** order and diverges on multi-axis rotations. Prefer `CFrame.fromOrientation` or explicit `CFrame.fromMatrix(pos, right, up)`.
- **Pitch Sign Around `+X`** (Right-Handed):
  - `CFrame.Angles(+theta, 0, 0)` rotates local `+Z` downward (`-Y`) and local `-Z` upward (`+Y`). Verify slope sign with corner transforms (`cf:PointToWorldSpace`) rather than guessing.
- **Axis-Size Parity Rule**:
  - When constructing a span between `p1` and `p2` along direction `dir = (p2 - p1)`:
    - If using `CFrame.lookAt(mid, p2)` (`LookVector = -Z` points along `dir`), put the span length in **`Size.Z`** (`Vector3.new(thickness, height, dist)`), **never** `Size.X` with an ad-hoc `* CFrame.Angles(0, rad(90), 0)` unless verified via basis vectors.
- **Pivots & Deprecated Constructors**:
  - Use `CFrame.lookAt(pos, target)` instead of deprecated `CFrame.new(pos, target)`.
  - Use `PVInstance:PivotTo(cf)` / `GetPivot()` and `Model:GetBoundingBox()` instead of deprecated `SetPrimaryPartCFrame()`.

---

## 4. Sealing, Light Bleeding, Z-Fighting & Performance
- **Voxel Light Bleed Prevention**:
  - Studio lighting evaluates 4×4×4 stud voxel grids alongside shadow maps (`LightingStyle` `Realistic`/`Soft` + `PrioritizeLightingQuality`, or legacy `Technology` `Voxel`/`ShadowMap`/`Future`).
  - Keep exterior walls and roofs **`>= 1` stud thick** (`1.5–2.0+` studs recommended), overlap corner junctions (`>= 0.25` stud) and wall-to-eave joins (`0.5–1.5` studs into roof underside), and overlap door infill piers `+0.2–0.4` studs into guide tracks so no sub-pixel sky slits remain.
  - Minimize shadow-casting local lights under `Future` lighting and offset ceiling/wall lights `~1` stud away from surfaces.
- **Z-Fighting Prevention**:
  - Never place coplanar visible faces at identical coordinates (e.g., `Workspace.Baseplate` top at `Y = 0` + asphalt slab top at `Y = 0` + interior floor tile bottom at `Y = 0`).
  - Offset floor overlays/lane markings (`+0.02–0.05` studs above base) and exterior fascia/trim (`+0.05–0.2` studs proud), maintaining `>= 0.005–0.01` stud minimum separation on parallel visible faces.
- **CSG vs Modular Parts vs `MeshPart` & Streaming**:
  - `GeometryService` (`UnionAsync`, `SubtractAsync`, `IntersectAsync`, `SweepPartAsync`, `FragmentAsync`) enforces a **20,000-triangle** limit per operation (`UsePartColor` inherits part colors) and creates unique non-instanced geometry that can suffer frustum-culling flicker.
  - For static architecture, prefer modular `Part`/`WedgePart` assemblies or instanced `MeshPart`s (`Enum.CollisionFidelity`: `Box` / `Hull` / `Default` / `PreciseConvexDecomposition` / `Tunable`; `Enum.RenderFidelity`: `Automatic` / `Precise` / `Performance`).
  - Keep static architecture `Anchored = true` and set `Model.ModelStreamingMode = Enum.ModelStreamingMode.Atomic` (or `Persistent` sparingly) on primary structures when `Workspace.StreamingEnabled = true`.
- **Materials**:
  - Use `MaterialVariant` (parented under `MaterialService`, assigned via `BasePart.Material` + `BasePart.MaterialVariant`, tiled via `StudsPerTile` where `8×8` is engine default / `3.57` studs ≈ 1 m) for repeatable architectural textures that preserve physics.
  - Use `SurfaceAppearance` (child of `MeshPart`, UV-mapped, immutable at runtime) for custom-unwrapped props.
