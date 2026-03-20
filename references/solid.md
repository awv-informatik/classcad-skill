# Solid API Reference — `api.v1.solid.*`

> Direct solid modeling within entity injection features: primitives, booleans, transforms, slicing, filleting, and cross-feature solid usage.

## Table of Contents

### Primitives
- [box](#box) — Create a box solid
- [sphere](#sphere) — Create a sphere solid
- [cylinder](#cylinder) — Create a cylinder solid
- [cone](#cone) — Create a cone solid

### Extrusion & Revolve
- [extrusion](#extrusion) — Extrude a shape or sketch geometry
- [revolve](#revolve) — Revolve a shape around an axis

### Boolean Operations
- [union](#union) — Union solids
- [subtraction](#subtraction) — Subtract tool solids from target
- [intersection](#intersection) — Intersect solids
- [merge](#merge) — Merge solids (NOT a boolean union)

### Transforms
- [translation](#translation) — Translate a solid by vector
- [rotation](#rotation) — Rotate a solid by rotation vector
- [mirror](#mirror) — Mirror a solid at a plane
- [scale](#scale) — Scale a solid by factor
- [offset](#offset) — Create offset shell of a solid
- [copy](#copy) — Copy a solid with optional transform

### Cutting
- [slice](#slice) — Cut solid at a plane
- [section](#section) — Section solid at a plane (returns curves)

### Filleting
- [fillet](#fillet) — Create fillet on B-rep edges

### Cross-feature
- [useSolid](#usesolid) — Use solids from other features in current entity injection
- [deleteSolid](#deletesolid) — Delete solids from entity injection

---

> **AGENT HINTS**:
> - **All solid operations require an entity injection feature ID** (`param.id`). Create one with `api.v1.part.entityInjection()` first.
> - **Boolean tools are consumed** by default. Set `keepTools: true` to preserve them.
> - **`offset()`** is fragile — only works on solids where all faces can be offset without topology changes.
> - **`useSolid()`** lets you reference solids from other features. Use `{ from: [featureId], in: eifId }`.
> - **Rotation order** in `rotation()`: Z first, then Y, then X.

---
<a name="deleteSolid"></a>

## deleteSolid(param)

Deletes the given solids or all solids if no solids are provided.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param       | Type                                                        | Description                                                     |
| ----------- | ----------------------------------------------------------- | --------------------------------------------------------------- |
| param       | <code>object</code>                                         | object containing all the parameters                            |
| param.id    | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to clear solids from         |
| [param.ids] | <code>Array&lt;(string\|real\|id)&gt;</code>                | ids of the solids to delete, if VOID all solids will be deleted |

**Example**

```js
api.v1.solid.deleteSolid({ id: entityInjectionFeature, ids: [solid1, solid, solid3] })
api.v1.solid.deleteSolid({ id: entityInjectionFeature })
```

> **AGENT NOTE (trained 2026-03-19):** Confirmed working. Specific deletion by ID: volume drops by exactly the deleted solid's volume. Delete-all (no `ids` param): clears all solids, mass properties return null after. Always returns VOID (null).

<a name="copy"></a>

## copy(param)

Creates a copy of the given solid

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the copy solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default           | Description                                                                                                                           |
| ------------------- | ----------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         |                   | object containing all the parameters                                                                                                  |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create copy in                                                                                  |
| param.target        | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the solid to copy                                                                                                               |
| [param.rotation]    | <code>point</code>                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                          |
| [param.translation] | <code>point</code>                                          |                   | translation vector along x, y and z-axis                                                                                              |
| [param.rotateFirst] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether the copied solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.copy({ id: entityInjectionFeature, target: solid })
api.v1.solid.copy({ id: entityInjectionFeature, target: solid, translation: [0, 0, 10], rotation: [0, 0, 1.57], rotateFirst: FALSE })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Creates a NEW solid:** Unlike translation/rotation/mirror/scale which modify in-place, `copy()` creates a new solid and returns its ID. The original solid is preserved unchanged.
>
> **Basic copy:** With no transform params, creates an exact duplicate at the same position. Total volume doubles.
>
> **Translation:** `copy({ ..., translation: [200,0,0] })` creates the copy offset by the given vector. Original stays put.
>
> **rotateFirst behavior:** Same pattern as primitive creation:
> - `rotateFirst: true` (default): rotate copy around origin, then translate. For a centered solid, rotation has no spatial effect, so copy ends up at the translation point.
> - `rotateFirst: false`: translate first, then rotate around world origin. Copy of centered solid translated to [200,0,0] then rotated 90° Z ends up at [0,200,0].
>
> **Use cases:** Mirror-copy pattern: `copy()` then `mirror()` on the copy. Linear pattern: loop with `copy()` + incremental translation.

<a name="box"></a>

## box(param)

Creates a box

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created box solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default           | Description                                                                                                                        |
| ------------------- | ----------------------------------------------------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         |                   | object containing all the parameters                                                                                               |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create the box solid in                                                                      |
| param.length        | <code>real</code>                                           |                   | length of the box in x-direction                                                                                                   |
| param.width         | <code>real</code>                                           |                   | width of the box in y-direction                                                                                                    |
| param.height        | <code>real</code>                                           |                   | height of the box in z-direction                                                                                                   |
| [param.rotation]    | <code>point</code>                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                       |
| [param.translation] | <code>point</code>                                          |                   | translation vector along x, y and z-axis                                                                                           |
| [param.rotateFirst] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether the box solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.box({ id: entityInjectionFeature, length: 150, width: 55, height: 258 })
api.v1.solid.box({ id: entityInjectionFeature, length: 150, width: 55, height: 258, translation: [0, 0, 150] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Centering:** Box is centered at the origin, spanning `[-L/2, -W/2, -H/2]` to `[L/2, W/2, H/2]`. This is NOT corner-aligned. To place a box with one corner at origin, use `translation: [L/2, W/2, H/2]`.
>
> **Volume:** Computed exactly as `length × width × height` — no floating-point deviation for typical values. Even thin plates (200×200×0.01 → volume 400) are exact.
>
> **BRep topology:** A box has **6 faces, 12 edges, 8 vertices**. Euler V-E+F = 2. `getGeometryPositions` returns exactly **4 sample points per face** (face center + cardinal directions). STL export produces exactly **12 triangles** (2 per face) for an unmodified box.
>
> **rotateFirst behavior:** Same general `solid.*` pattern as all primitives:
> - `rotateFirst: true` (default): rotate around origin, then translate → CoG ends up at translation point.
> - `rotateFirst: false`: translate first, then rotate around world origin → final position = rotated translation vector.
> Example: `translation=[100,0,0]` + `rotation=[0,0,π/2]` with `rotateFirst=false` → CoG at `[0,100,0]`.
>
> **Edge cases — zero/negative dimensions:**
> - `height: 0` — box is created (returns a valid ID) but produces a degenerate solid. No error or warning is raised. Avoid.
> - `height: -40` — box is created but `calculateMassProperties` returns `undefined` volume. The solid is broken. **Always use positive dimensions.**
>
> **Multi-solid BRep:** When multiple boxes exist in the same EIF, `getBrepGeometryByIndex` without `solidIndex` only returns the first solid's elements. Pass `solidIndex: 0`, `1`, etc. to enumerate each solid separately. Each box contributes its own 6 faces, 12 edges, 8 vertices.

<a name="sphere"></a>

## sphere(param)

Creates a sphere

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created sphere solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default           | Description                                                                                                                           |
| ------------------- | ----------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         |                   | object containing all the parameters                                                                                                  |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create the sphere solid in                                                                      |
| param.radius        | <code>real</code>                                           |                   | radius of the sphere                                                                                                                  |
| [param.rotation]    | <code>point</code>                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                          |
| [param.translation] | <code>point</code>                                          |                   | translation vector along x, y and z-axis                                                                                              |
| [param.rotateFirst] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether the sphere solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.sphere({ id: entityInjectionFeature, radius: 60 })
api.v1.solid.sphere({ id: entityInjectionFeature, radius: 60, rotation: [1.57, 0, 0], translation: [0, 50, 0], rotateFirst: FALSE })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Centering:** Sphere is created centered at the origin. Use `translation` to place it elsewhere. CoG of a translated sphere exactly matches the translation vector.
>
> **Volume precision:** Volume matches `(4/3)πr³` to within ~0.014% for typical radii (r=50). For very large spheres (r=10000) the error is ~0.006%. This is analytic precision, not mesh-dependent.
>
> **BRep topology:** A single sphere has **1 face, 0 edges, 2 vertices** (the pole vertices). Euler characteristic V-E+F = 3 (not 2) — this is because the sphere surface is a single face bounded by no edges, with two degenerate pole points. When enumerating BRep with `getBrepGeometryByIndex`, use `solidIndex` parameter when multiple solids exist in the same EIF, otherwise only the first solid's BRep is returned.
>
> **rotateFirst behavior:** When both `rotation` and `translation` are provided:
> - `rotateFirst: true` (default): rotation is applied first (no spatial effect on a sphere), then translation. CoG ends up at the translation point.
> - `rotateFirst: false`: translation is applied first (sphere moves to translation point), then the entire body is rotated around the **world origin**. For example, with `translation=[100,0,0]` and `rotation=[0,0,π/2]`, the CoG ends up at `[0,100,0]` — the translation vector is rotated.
> This distinction is critical for non-symmetric solids.
>
> **Tiny spheres:** `calculateMassProperties` returns `undefined` for very small spheres (r=0.01). The solid is created successfully, but mass property computation fails silently. Avoid radii below ~0.1 for reliable mass queries.
>
> **Multiple spheres in one EIF:** Mass properties correctly sum across all solids. CoG is the volume-weighted average of individual centers.

<a name="cylinder"></a>

## cylinder(param)

Creates a cylinder

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created cylinder solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default           | Description                                                                                                                             |
| ------------------- | ----------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         |                   | object containing all the parameters                                                                                                    |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create the cylinder solid in                                                                      |
| param.height        | <code>real</code>                                           |                   | height of the cylinder                                                                                                                  |
| param.diameter      | <code>real</code>                                           |                   | diameter of the cylinder                                                                                                                |
| [param.rotation]    | <code>point</code>                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                            |
| [param.translation] | <code>point</code>                                          |                   | translation vector along x, y and z-axis                                                                                                |
| [param.rotateFirst] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether the cylinder solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.cylinder({ id: entityInjectionFeature, height: 300, diameter: 50 })
api.v1.solid.cylinder({ id: entityInjectionFeature, height: 50, diameter: 50, translation: [50, 0, 0] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Critical: parameter is `diameter`, NOT `radius`.** Unlike `solid.sphere` which takes `radius`, cylinder takes `diameter`. A cylinder with `diameter: 100` has radius 50. This was verified by comparing computed volume against both interpretations. This inconsistency is a common source of bugs.
>
> **Centering & axis:** Cylinder is centered at origin both radially (XY plane) and vertically (Z axis). A cylinder with `height: 100` spans Z=[-50, +50]. **Default axis is Z.** To lay flat along X: `rotation: [0, π/2, 0]`. To lay along Y: `rotation: [π/2, 0, 0]`.
>
> **Volume precision:** Typically ~0.001% error vs analytic πr²h. **Exception: extreme aspect ratios** — a tall thin cylinder (h=500, d=2) showed **~1% error**, much higher than box or sphere. The flat disc (h=0.1, d=200) was accurate. Be cautious with very small diameters.
>
> **BRep topology:** A cylinder has **3 faces, 1 edge, 2 vertices**. Euler V-E+F = 4. The faces are: bottom cap (1 sample point at Z=-h/2), lateral surface (3 sample points at avgZ=0), top cap (1 sample point at Z=+h/2). The single edge is the shared circular boundary. The 2 vertices are pole/seam points.
>
> **rotateFirst behavior:** Same general `solid.*` pattern. With `translation=[100,0,0]` + `rotation=[0,0,π/2]`: `rotateFirst=true` → CoG at [100,0,0]; `rotateFirst=false` → CoG at [0,100,0].
>
> **Edge cases:**
> - `diameter: 0` — creates a degenerate solid silently (no error). Avoid.
> - `diameter: -60` — creates a solid but with near-zero volume (0.094). Broken. **Always use positive diameter.**
> - `height: 0` — creates a degenerate solid silently. Avoid.

<a name="cone"></a>

## cone(param)

Creates a cone

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created cone solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default           | Description                                                                                                                         |
| ------------------- | ----------------------------------------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         |                   | object containing all the parameters                                                                                                |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create the cone solid in                                                                      |
| param.height        | <code>real</code>                                           |                   | height of the cone                                                                                                                  |
| param.bDiameter     | <code>real</code>                                           |                   | diameter at the bottom of the cone                                                                                                  |
| param.tDiameter     | <code>real</code>                                           |                   | diameter at the top of the cone                                                                                                     |
| [param.rotation]    | <code>point</code>                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                        |
| [param.translation] | <code>point</code>                                          |                   | translation vector along x, y and z-axis                                                                                            |
| [param.rotateFirst] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether the cone solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.cone({ id: entityInjectionFeature, height: 300, bDiameter: 50, tDiameter: 0.1 })
api.v1.solid.cone({ id: entityInjectionFeature, height: 300, bDiameter: 50, tDiameter: 0.1, rotation: [0, 0, 3.1415] })
api.v1.solid.cone({ id: entityInjectionFeature, height: 300, bDiameter: 50, tDiameter: 0.1, translation: [0, 0, 10] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Parameters are diameters, not radii.** `bDiameter` and `tDiameter` are full diameters (consistent with `solid.cylinder`, inconsistent with `solid.sphere` which uses `radius`). Verified by volume formula match: a cone with `bDiameter: 60, tDiameter: 20, height: 100` produces volume matching `(π·h/3)·(R² + R·r + r²)` where R=bD/2, r=tD/2 to within 0.0004%.
>
> **Centering & axis:** Cone is centered at origin along Z axis. Bottom face (bDiameter end) at Z=-h/2, top face (tDiameter end) at Z=+h/2. "b" means bottom (Z=-h/2), "t" means top (Z=+h/2) — geometrically, NOT "big"/"tiny".
>
> **Cone variants:**
> - **Frustum:** bDiameter ≠ tDiameter (the general case)
> - **True cone:** tDiameter = 0 (or near-zero, e.g. 0.1 as in the docs example). Volume matches `(1/3)·π·R²·h`.
> - **Cylinder-equivalent:** bDiameter = tDiameter. Volume matches `π·r²·h` to within 0.0009%.
> - **Inverted cone:** tDiameter > bDiameter. Works correctly — the bDiameter end stays at Z=-h/2 regardless.
>
> **BRep topology:** Always **3 faces, 1 edge, 2 vertices**, Euler V-E+F = 4. Same topology as cylinder. Even when `tDiameter = 0`, the kernel keeps a degenerate top cap face. Face identification via `getGeometryPositions` avgZ: face at avgZ=-h/2 = bottom cap, avgZ=0 = lateral surface, avgZ=+h/2 = top cap.
>
> **Center of gravity (CoG):** For asymmetric frustums, CoG is NOT at Z=0. It shifts toward the heavier (larger diameter) end. Formula: `CoG_z = -h/2 + h·(R² + 2Rr + 3r²) / (4·(R² + Rr + r²))`. Verified against ClassCAD output (e.g., bD=60, tD=20, h=100 → CoG_z = -15.384).
>
> **Volume precision:** ~0.0004% for normal proportions. Degrades to **~1.5% for extreme aspect ratios** (h=500, bD=2, tD=0.1). Same precision pattern as cylinder. Flat frustum (h=10, bD=200, tD=100) maintains good precision (~0.001%).
>
> **rotateFirst behavior:** Same universal `solid.*` pattern confirmed for 4th primitive.
> - `rotateFirst: true` (default): rotate → translate. Example: `translation=[100,0,0]` + `rotation=[0,0,π/2]` → CoG at [100, 0, CoG_z].
> - `rotateFirst: false`: translate → rotate around world origin → CoG at [0, 100, CoG_z].
>
> **Edge cases — degenerate inputs:**
> - `tDiameter: 0` — accepted, creates valid cone. Volume matches analytic formula. Docs example uses 0.1 to avoid this edge.
> - `bDiameter: 0` — accepted silently, creates degenerate solid. Avoid.
> - Negative `bDiameter` — accepted (returns valid ID), but `calculateMassProperties` throws "Division by zero!". **Always use positive values.**
> - `height: 0` — accepted silently, degenerate solid. Avoid.
>
> **Multi-cone export:** Multiple cones in same EIF export correctly to STL/STEP/OFB. Use `solidIndex` for BRep enumeration of individual cones.

<a name="extrusion"></a>

## extrusion(param)

Creates an extrusion by extruding a shape or sketch geometry

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created extrusion solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                                                                        | Default           | Description                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                                                                         |                   | object containing all the parameters                                                                                                    |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                 |                   | id of the entity injection feature to create the extrusion in                                                                           |
| param.direction     | <code>point</code>                                                                                          |                   | direction of the extrusion, includes the distance as well                                                                               |
| param.curves        | <code>Array&lt;(string\|real\|id)&gt;</code> \| <code>string</code> \| <code>real</code> \| <code>id</code> |                   | either an array of sketch element ids or a shape id, which is a container for curves created with the curveAPI                          |
| [param.rotation]    | <code>point</code>                                                                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                            |
| [param.translation] | <code>point</code>                                                                                          |                   | translation vector along x, y and z-axis                                                                                                |
| [param.rotateFirst] | <code>boolean</code>                                                                                        | <code>TRUE</code> | flag to define whether the extruded solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.extrusion({ id: entityInjectionFeature, direction: [0, 0, 120], curves: shape })
api.v1.solid.extrusion({ id: entityInjectionFeature, direction: [0, 0, 120], curves: shape, rotation: [0, 0, 1.57] })
```

> **AGENT NOTE (trained 2026-03-19):** Extrusion works perfectly. Volume = profile_area × |direction|. `rotateFirst=true` (default): rotate around origin then translate → CoG at translated position. `rotateFirst=false`: translate first then rotate around world origin → CoG rotated around origin. The `curves` param takes a shape ID from `v1.curve.shape` containing polyline2d profiles. Profile points must be `Array<point>` format `[[x,y,z],...]`.

<a name="revolve"></a>

## revolve(param)

Creates a revolve by revolving a polyline

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created revolve solid
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                                                                        | Default           | Description                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                                                                         |                   | object containing all the parameters                                                                                                    |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                 |                   | id of the entity injection feature to create the revolve in                                                                             |
| param.originPos     | <code>point</code>                                                                                          |                   | origin position of the rotation axis                                                                                                    |
| param.direction     | <code>point</code>                                                                                          |                   | direction of the rotation axis                                                                                                          |
| param.angle         | <code>real</code>                                                                                           |                   | rotation angle of the revolve in radians                                                                                                |
| param.curves        | <code>Array&lt;(string\|real\|id)&gt;</code> \| <code>string</code> \| <code>real</code> \| <code>id</code> |                   | either an array of sketch element ids or a shape id, which is a container for curves created with the curveAPI                          |
| [param.rotation]    | <code>point</code>                                                                                          |                   | rotation vector containing rotations around x, y, and z-axis                                                                            |
| [param.translation] | <code>point</code>                                                                                          |                   | translation vector along x, y and z-axis                                                                                                |
| [param.rotateFirst] | <code>boolean</code>                                                                                        | <code>TRUE</code> | flag to define whether the revolved solid should be first rotated or not, when both rotation and translation is provided (default=TRUE) |

**Example**

```js
api.v1.solid.revolve({ id: entityInjectionFeature, direction: [0, 1, 0], originPos: [0, 0, 0], angle: 3.14, curves: shape })
api.v1.solid.revolve({ id: entityInjectionFeature, direction: [0, 1, 0], originPos: [0, 0, 0], angle: 3.14, curves: shape, translation: [20, 0, 20] })
```

> **AGENT NOTE (trained 2026-03-19):** Revolve validated — volume matches Pappus theorem (V = 2πRA for full revolution). Angle in radians: 2π = full, π = half. **Critical: profile must NOT intersect the revolution axis** or you get "Brep after revolve operation not manifold" error. Offset the profile away from the axis.

<a name="intersection"></a>

## intersection(param)

Creates an intersection between solids

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Default            | Description                                                                  |
| ----------------- | ----------------------------------------------------------- | ------------------ | ---------------------------------------------------------------------------- |
| param             | <code>object</code>                                         |                    | object containing all the parameters                                         |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id of the entity injection feature to create the intersection in             |
| param.target      | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | solid to use as base for this boolean operation                              |
| param.tools       | <code>Array&lt;(string\|real\|id)&gt;</code>                |                    | solids to use as tools for this boolean operation                            |
| [param.keepTools] | <code>boolean</code>                                        | <code>FALSE</code> | flag to define whether the tool solids should be kept or not (default=FALSE) |

**Example**

```js
api.v1.solid.intersection({ id: entityInjectionFeature, target: solid, tools: [solid2] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Semantics:** Computes the volume shared by `target` and all `tools`. Result replaces the target (returned ID = target ID).
>
> **Volume accuracy:** Two overlapping 100³ boxes with 50 overlap → intersection volume = 500000 (exact). CoG correctly at center of overlap region.
>
> **Non-overlapping solids:** Returns `null` (VOID). Message: `[level 51] Target solid was removed by intersection.` Same behavior as full subtraction — `calculateMassProperties` throws after this.
>
> **keepTools:** `false` (default) consumes tools. `true` preserves them. With keepTools=true, total volume = intersection + preserved tool(s).
>
> **Return value:** Modified target ID on success, `null` if no overlap exists.

<a name="subtraction"></a>

## subtraction(param)

Creates a subtraction between solids

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Default            | Description                                                                  |
| ----------------- | ----------------------------------------------------------- | ------------------ | ---------------------------------------------------------------------------- |
| param             | <code>object</code>                                         |                    | object containing all the parameters                                         |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id of the entity injection feature to create the box solid in                |
| param.target      | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | solid to use as base for this boolean operation                              |
| param.tools       | <code>Array&lt;(string\|real\|id)&gt;</code>                |                    | solids to use as tools for this boolean operation                            |
| [param.keepTools] | <code>boolean</code>                                        | <code>FALSE</code> | flag to define whether the tool solids should be kept or not (default=FALSE) |

**Example**

```js
api.v1.solid.subtraction({ id: entityInjectionFeature, target: solid, tools: [solid2] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Semantics:** Subtracts all `tools` from `target`. The result replaces the target solid (returned ID = target ID). Tool geometry is removed from the target's volume.
>
> **Volume accuracy:** Box minus through-hole cylinder: ~0.0004% error vs analytic formula. Subtraction correctly clips the tool to the target's bounds.
>
> **Full subtraction (tool engulfs target):** Returns `null` (VOID). Message: `[level 51] Target solid was removed by subtraction.` After this, `calculateMassProperties` throws "Division by zero!" — no solids remain.
>
> **keepTools:** `false` (default) consumes tools. `true` preserves them as separate solids.
>
> **Return value:** Modified target ID on success, `null` if target was completely consumed.

<a name="union"></a>

## union(param)

Creates a union between solids

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Default            | Description                                                                  |
| ----------------- | ----------------------------------------------------------- | ------------------ | ---------------------------------------------------------------------------- |
| param             | <code>object</code>                                         |                    | object containing all the parameters                                         |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id of the entity injection feature to create the box solid in                |
| param.target      | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | solid to use as base for this boolean operation                              |
| param.tools       | <code>Array&lt;(string\|real\|id)&gt;</code>                |                    | solids to use as tools for this boolean operation                            |
| [param.keepTools] | <code>boolean</code>                                        | <code>FALSE</code> | flag to define whether the tool solids should be kept or not (default=FALSE) |

**Example**

```js
api.v1.solid.union({ id: entityInjectionFeature, target: solid, tools: [solid1, solid2], keepTools: TRUE })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Semantics:** Union computes the boolean union of `target` with all `tools`. The result replaces the target solid — the returned ID equals the target ID. Overlapping volume is removed (not double-counted).
>
> **Volume accuracy:** Exact for box-box unions (0.0000% error). For box-cylinder unions, ~0.0004% error (kernel tessellation).
>
> **keepTools behavior:**
> - `keepTools: false` (default): tool solids are **consumed** — they no longer exist in the EIF. Only the union result remains (1 solid).
> - `keepTools: true`: tool solids are **preserved** as separate solids alongside the union result. Volume reported by `calculateMassProperties` includes both the union result AND the preserved tools (double-counting the overlap region).
>
> **Multiple tools:** Passing multiple tool IDs in `tools: [id1, id2, ...]` performs union with all tools in a single call. Inclusion-exclusion applies correctly — overlapping regions between tools are handled. Verified with 3 overlapping boxes.
>
> **BRep after union:** Two overlapping 100³ boxes (50 overlap in X) produce a union with **6 faces, 12 edges, 8 vertices** — the kernel simplifies coplanar faces. Euler V-E+F = 2.
>
> **Coincident faces:** Union of two boxes sharing an exact face boundary (touching, no overlap) succeeds. Volume = sum of both boxes. Kernel handles this edge case without errors.
>
> **Chained booleans:** The returned ID from union can be used as `target` in subsequent boolean operations. Pipeline: `union → subtraction` works correctly.
>
> **Return value:** Returns the (modified) target solid ID on success. If the union somehow produces nothing, returns VOID/null.

<a name="mirror"></a>

## mirror(param)

Mirrors the given solid at defined plane

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param           | Type                                                        | Description                                                |
| --------------- | ----------------------------------------------------------- | ---------------------------------------------------------- |
| param           | <code>object</code>                                         | object containing all the parameters                       |
| param.id        | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the mirror in |
| param.target    | <code>string</code> \| <code>real</code> \| <code>id</code> | solid to mirror                                            |
| param.originPos | <code>point</code>                                          | origin position of the plane to mirror at                  |
| param.normal    | <code>point</code>                                          | normal direction of the plane to mirror at                 |

**Example**

```js
api.v1.solid.mirror({ id: entityInjectionFeature, target: solid, originPos: [50, 0, 0], normal: [0, 0, 1] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Mirror modifies the target solid in-place. The returned ID equals the target ID. **No new solid is created** — this is NOT a "mirror copy." The original solid is reflected to the new position.
>
> **Volume preserved:** Volume is exactly unchanged.
>
> **Plane definition:** `originPos` is a point on the mirror plane, `normal` is the plane's normal direction. The solid is reflected across this plane.
>
> **Example:** A box centered at [100,0,0] mirrored at the YZ plane (originPos=[0,0,0], normal=[1,0,0]) ends up at [-100,0,0].
>
> **Offset plane:** Mirroring at X=100 (originPos=[100,0,0], normal=[1,0,0]) moves a solid from [50,0,0] to [150,0,0] (reflected through the plane).
>
> **To create a mirror copy:** Use `copy()` first, then `mirror()` on the copy. Or create the solid twice.

<a name="translation"></a>

## translation(param)

Translates the given solid by the given vector. The vector is in coordinates of the part
where the provided solid belongs to.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Description                                                     |
| ----------------- | ----------------------------------------------------------- | --------------------------------------------------------------- |
| param             | <code>object</code>                                         | object containing all the parameters                            |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the translation in |
| param.target      | <code>string</code> \| <code>real</code> \| <code>id</code> | solid to translate                                              |
| param.translation | <code>point</code>                                          | translation vector along x, y and z-axis                        |

**Example**

```js
api.v1.solid.translation({ id: entityInjectionFeature, target: solid, translation: [0, 0, 85] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Translation modifies the target solid in-place. The returned ID equals the target ID. No new solid is created.
>
> **Volume preserved:** Volume is exactly unchanged after translation.
>
> **Cumulative:** Multiple translations on the same solid are additive. Translating by [50,0,0] then [0,30,0] results in a net displacement of [50,30,0].
>
> **Coordinate system:** The translation vector is in part coordinates. For solids in an EIF, this is the global coordinate system unless the part itself has been repositioned.

<a name="offset"></a>

## offset(param)

Creates an offset solid of the given solid

Note: This functionality is a quite difficult and fragile one. Make sure
you call it only on solids, where you can offset all faces, calculate the new trims AND
dont change the topology of the solid i.e. the number of faces and edges remain the same.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param          | Type                                                        | Default            | Description                                                                                                                                                                  |
| -------------- | ----------------------------------------------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param          | <code>object</code>                                         |                    | object containing all the parameters                                                                                                                                         |
| param.id       | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id of the entity injection feature to create the offset in                                                                                                                   |
| param.target   | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | solid to create an offset from                                                                                                                                               |
| param.distance | <code>real</code>                                           |                    | distance of the offset                                                                                                                                                       |
| [param.extend] | <code>boolean</code>                                        | <code>FALSE</code> | extend means that surfaces are extended beyound their current trimming curves (default=FALSE). If FALSE the algorithm fills the gap with fillets with radius equal distance. |

**Example**

```js
api.v1.solid.offset({ id: entityInjectionFeature, target: solid, distance: 10 })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Offset modifies the target solid in-place. Returns the target ID.
>
> **Positive distance (outward):** Expands the solid outward. With `extend: false` (default), fillets are added at edges with radius = distance. For a 100×60×40 box with distance=5, volume = 380231 (not 385000) because of the rounded edges/corners.
>
> **extend=true:** Faces are extended beyond trim curves without fillets. For a box, this produces exact rectangular growth: 100×60×40 + distance=5 → 110×70×50 = 385000 (exact). **Use `extend: true` for boxes/flat-faced solids when you want precise dimensional growth.**
>
> **Negative distance (inward):** Shrinks the solid. However, for a 100×60×40 box with distance=-5, mass properties returned **null** — the operation likely failed silently or produced an invalid solid. The docs warn this is fragile. **Be cautious with negative offsets; test on your specific geometry.**
>
> **Fragility warning confirmed:** As documented, offset can fail on solids where face offsetting changes topology. Negative offsets on boxes fail. Complex geometry is even more likely to fail.

<a name="rotation"></a>

## rotation(param)

Rotates the given solid by the given rotation vector. The vector is in coordinates of the part
where the provided solid belongs to.
First the z-part of the rotation vector is performed, then the y-part and finally the x-part.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param          | Type                                                        | Description                                                  |
| -------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| param          | <code>object</code>                                         | object containing all the parameters                         |
| param.id       | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the rotation in |
| param.target   | <code>string</code> \| <code>real</code> \| <code>id</code> | solid to rotate                                              |
| param.rotation | <code>point</code>                                          | rotation vector containing rotations around x, y, and z-axis |

**Example**

```js
api.v1.solid.rotation({ id: entityInjectionFeature, target: solid, rotation: [3.14, 0, 0] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Rotation modifies the target solid in-place. Returns the target ID.
>
> **Rotation center:** Rotates around the **world origin** (part coordinate origin), NOT around the solid's center. A solid at [100,0,0] rotated 90° around Z ends up at [0,100,0].
>
> **Rotation order confirmed:** Z first, then Y, then X. With rotation=[π/2, 0, π/2]: a point at [100,0,0] → Z rotation → [0,100,0] → X rotation → [0,0,100]. This matches the documented Z→Y→X order.
>
> **Volume preserved:** Volume is unchanged (within floating-point noise, ~1e-11 relative error).
>
> **180° rotation:** Correctly inverts the appropriate coordinate signs. A solid at [50,30,20] rotated [0,0,π] ends up at [-50,-30,20].

<a name="scale"></a>

## scale(param)

Scales the given solid with a factor. The scale is in coordinates of the part
where the provided solid belongs to.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                                                        | Description                                               |
| ------------ | ----------------------------------------------------------- | --------------------------------------------------------- |
| param        | <code>object</code>                                         | object containing all the parameters                      |
| param.id     | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the scale in |
| param.target | <code>string</code> \| <code>real</code> \| <code>id</code> | solid to scale                                            |
| param.factor | <code>real</code>                                           | scale factor of the solid                                 |

**Example**

```js
api.v1.solid.scale({ id: entityInjectionFeature, target: solid, factor: 2.5 })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Scale modifies the target solid in-place. Returns the target ID.
>
> **Scale center:** Scales around the **world origin** (part coordinate origin), NOT around the solid's center. A solid at [100,0,0] scaled by 2 will have its CoG at [200,0,0]. A centered solid stays centered.
>
> **Volume scaling:** Volume scales by factor³. factor=2 → 8× volume. factor=0.5 → 0.125× volume. Exact for integer factors.
>
> **Identity:** factor=1 is an exact identity — no floating-point drift.
>
> **Negative factor:** factor=-1 is accepted. It produces a **negative volume** (-240000 for a 240000 box) and mirrors the solid through the origin. The CoG is negated. This is a valid geometric reflection but the negative volume may cause issues downstream. **Avoid negative scale factors** — use `mirror()` instead.
>
> **Uniform only:** There is no non-uniform scaling (per-axis). The single `factor` applies equally in all directions.

<a name="slice"></a>

## slice(param)

Cuts the given solid at defined plane. The part on the negative
side of normal vector is removed

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID  // id of the slice if keepBoth flag is TRUE
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param            | Type                                                        | Default           | Description                                                             |
| ---------------- | ----------------------------------------------------------- | ----------------- | ----------------------------------------------------------------------- |
| param            | <code>object</code>                                         |                   | object containing all the parameters                                    |
| param.id         | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | id of the entity injection feature to create the sliced solid in        |
| param.target     | <code>string</code> \| <code>real</code> \| <code>id</code> |                   | solid to slice                                                          |
| param.originPos  | <code>point</code>                                          |                   | origin position of the plane to cut at                                  |
| param.normal     | <code>point</code>                                          |                   | normal direction of the plane to cut at                                 |
| [param.keepBoth] | <code>boolean</code>                                        | <code>TRUE</code> | flag to define whether both solids should be kept or not (default=TRUE) |

**Example**

```js
api.v1.solid.slice({ id: entityInjectionFeature, target: solid, originPos: [0, 0, 50], normal: [0, 1, 1], keepBoth: FALSE })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **⚠️ DOCS INVERSION: The POSITIVE side of the normal is removed, not the negative side.** The docs say "The part on the negative side of normal vector is removed" — this is WRONG. Testing proves: `normal=[0,0,1]` with `keepBoth=false` keeps the Z<0 half (CoG at Z=-25), removing the Z>0 (positive) half. Flipping to `normal=[0,0,-1]` keeps Z>0 (CoG at Z=+25). **The half on the positive side of the normal is discarded.**
>
> **keepBoth=true (default):** Both halves are preserved as separate solids. Total volume is unchanged. The returned ID is the **new** solid (the piece that would otherwise be removed). The original target solid is modified in-place to become the "kept" half.
>
> **keepBoth=false:** Only one half remains. Returns `null` (not the surviving solid ID — the original target ID is still valid). Volume = half for midplane cuts.
>
> **Plane definition:** `originPos` = a point on the cutting plane. `normal` = plane normal direction.
>
> **Plane outside solid:** If the cutting plane doesn't intersect the solid, the solid is unchanged. Returns `null`. No error or warning.
>
> **Angled cuts:** Work correctly. `normal=[1,0,1]` produces a diagonal cut. Total volume preserved with keepBoth=true.
>
> **Volume precision:** Exact for planar cuts (500000.0 for midplane of 1e6 box, within floating-point noise).

<a name="section"></a>

## section(param)

The given solid is sectioned at the given plane.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created section array
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param           | Type                                                        | Description                                              |
| --------------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| param           | <code>object</code>                                         | object containing all the parameters                     |
| param.id        | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the section |
| param.target    | <code>string</code> \| <code>real</code> \| <code>id</code> | solid to create section in                               |
| param.originPos | <code>point</code>                                          | origin position of the plane to cut at                   |
| param.normal    | <code>point</code>                                          | normal direction of the plane to cut at                  |

**Example**

```js
api.v1.solid.section({ id: entityInjectionFeature, target: solid, originPos: [0, 0, 120], normal: [0, 0, 1] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **Non-destructive:** Section does NOT modify the target solid. Volume is unchanged after the call.
>
> **Returns a single ID** (not an array): The returned ID appears to be a section/shape container. For a box section at Z=0, returns a single number. For a cylinder section, also a single number.
>
> **Plane outside solid:** Returns an ID even when the plane doesn't intersect the solid — the section container may be empty. No error or warning raised.
>
> **Use case:** Extract 2D cross-section curves from a 3D solid for analysis, drawing, or projection. The returned curves can potentially be used with curve/drawing APIs.

<a name="merge"></a>

## merge(param)

Creates an merge between solids. This is NOT a union!

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Default            | Description                                                                  |
| ----------------- | ----------------------------------------------------------- | ------------------ | ---------------------------------------------------------------------------- |
| param             | <code>object</code>                                         |                    | object containing all the parameters                                         |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id of the entity injection feature to create the box solid in                |
| param.target      | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | solid to use as base for this merge operation                                |
| param.tools       | <code>Array&lt;(string\|real\|id)&gt;</code>                |                    | solids to use as tools for this merge operation                              |
| [param.keepTools] | <code>boolean</code>                                        | <code>FALSE</code> | flag to define whether the tool solids should be kept or not (default=FALSE) |

**Example**

```js
api.v1.solid.merge({ id: entityInjectionFeature, target: solid, tools: [solid1, solid2], keepTools: TRUE })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **THIS IS NOT A BOOLEAN UNION.** Merge groups tool solids into the target without computing any boolean operation. Overlapping regions are NOT removed — volume is the raw sum of all individual solid volumes.
>
> **Merge vs Union (same overlapping 80³ boxes, 40 offset):** Union volume = 768000 (overlap removed). Merge volume = 1024000 (= 2 × 512000, no overlap removal). **Use union for material combination, merge for grouping.**
>
> **Volume unchanged:** Before and after merge, `calculateMassProperties` returns the same total volume (2000000 for two 100³ boxes). The merge operation absorbs tools into the target but doesn't modify geometry.
>
> **keepTools:** `false` (default) consumes tool solids (they merge into target, 1 solid remains). `true` preserves tools as separate solids (2 solids: merged target + original tool).
>
> **Use case:** Merge is for combining separate bodies into a single compound solid without modifying their geometry. Think of it as "group" not "fuse."

<a name="fillet"></a>

## fillet(param)

Creates a fillet at the given edges. Edges can be of different solids.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id[] | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param         | Type                                                        | Description                                                |
| ------------- | ----------------------------------------------------------- | ---------------------------------------------------------- |
| param         | <code>object</code>                                         | object containing all the parameters                       |
| param.id      | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the entity injection feature to create the fillet in |
| param.radius  | <code>real</code>                                           | radius of the fillet                                       |
| param.geomIds | <code>Array&lt;(string\|real\|id)&gt;</code>                | ids of the brep edges to create the fillet on              |

**Example**

```js
api.v1.solid.fillet({ id: entityInjectionFeature, radius: 5, geomIds: [85, 89, 93, 97] })
```

> **AGENT NOTE — verified by live testing (2026-03-19):**
>
> **In-place modification:** Fillet modifies the target solid(s) in-place. Returns an array containing the solid ID(s) that were modified (e.g., `[solidId]`).
>
> **geomIds are BRep edge IDs:** Get them via `getBrepGeometryByIndex` with `lineIndex`. A standard box has 12 edges.
>
> **Volume reduction:** Filleting removes material at edges. A 100³ box with r=10 fillet on 4 edges loses ~0.82% volume. All 12 edges filleted: ~2.44% volume reduction.
>
> **BRep change:** Each filleted edge adds a new cylindrical face and splits adjacent faces. 4 filleted edges on a box: 6F→10F, 12E→16E, 8V→12V. All 12 edges: 26F/24E/24V.
>
> **Radius too large:** If the fillet radius is too large for the geometry, the operation **silently does nothing** — returns `[solidId]` but volume is unchanged (still 1e6). No error or warning. **Always verify volume changed if you expect it to.**
>
> **Cylinder filleting:** A cylinder has 1 BRep edge (the circular seam). Filleting this edge with r=5 succeeds (returns solid ID) but volume is unchanged — the edge may be degenerate or the fillet is tangent to existing surfaces.
>
> **Cross-solid filleting:** Docs say "Edges can be of different solids" — pass edge IDs from multiple solids in the same `geomIds` array.
>
> **Workflow:** Fillet works on sliced solids. Slice → get edges → fillet is a valid pipeline.

<a name="useSolid"></a>

## useSolid(param)

This method allows you to work with solids from other features. The solids from the provided feature will
be returned and available to use within the given entity injection feature.

**Kind**: v1.solid function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id[] | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                             | Description                                                                                                             |
| ---------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| param                  | <code>object</code>                                                              | object containing all the parameters                                                                                    |
| param.from             | <code>Array&lt;(string\|real\|id)&gt;</code> \| <code>Array&lt;object&gt;</code> | features as ids to get solids from or as objects containing an id and optional indices                                  |
| param.from[].id        | <code>string</code> \| <code>real</code> \| <code>id</code>                      | id of the feature to get solids from                                                                                    |
| [param.from[].indices] | <code>Array&lt;real&gt;</code>                                                   | if more than one solid is appended to the feature, the indices can be used to specifiy which solids to get from feature |
| param.in               | <code>string</code> \| <code>real</code> \| <code>id</code>                      | id of the entity injection feature to use the solids in                                                                 |

**Example**

```js
api.v1.solid.useSolid({ from: [feature, feature2], in: entityInjectionFeature })
api.v1.solid.useSolid({ from: [{ id: feature, indices: [0, 1] }], in: entityInjectionFeature })
api.v1.solid.useSolid({ from: [{ id: feature, indices: [0, 1] }, { id: feature2 }], in: entityInjectionFeature })
```

> **AGENT NOTE (trained 2026-03-19):** Returns array of new solid IDs in the target EIF. Cross-feature referencing confirmed working — can reference from multiple EIFs. Selective indices work: `indices: [0]` picks only the first solid. Referenced solids are fully operable (boolean, transform, etc.). **Gotcha: source EIF must still contain solids.** If the source was consumed by a failed operation or has no geometry, useSolid returns `[]` with error "Entity...is not available. It has already been consumed/used in another operation."
