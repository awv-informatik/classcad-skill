# Assembly API Reference — `api.v1.assembly.*`

> Assembly building: templates, instances, constraints (fastened, revolute, cylindrical, parallel, planar, slider, spherical, gear, group), patterns (linear, circular), and assembly management.

## Table of Contents

### Templates & Instances
- [partTemplate](#parttemplate) — Create a new part template
- [assemblyTemplate](#assemblytemplate) — Create a new assembly template
- [instance](#instance) — Create instances of products
- [deleteInstance](#deleteinstance) — Delete instances
- [getPartTemplate](#getparttemplate) — Get part template(s) by name
- [getAssemblyTemplate](#getassemblytemplate) — Get assembly template(s) by name
- [getInstance](#getinstance) — Get instance(s) by owner and name
- [convertToTemplate](#converttotemplate) — Convert instance to template

### Assembly Lifecycle
- [create](#create) — Create a root assembly
- [loadProduct](#loadproduct) — Load a product into the assembly
- [exportNode](#exportnode) — Export an assembly node
- [setCurrentInstance](#setcurrentinstance) — Set the active instance
- [setCurrentProduct](#setcurrentproduct) — Set the active product
- [setIdent](#setident) — Set custom identifier
- [from](#from) — Create assembly from existing data
- [createUncommitedObject](#createuncommitedobject) — Create uncommitted object
- [deleteTemplate](#deletetemplate) — Delete a template
- [getWorkGeometry](#getworkgeometry) — Get work geometry from assembly

### Fastened Constraints
- [fastened](#fastened) — Create fastened constraint (fully locked)
- [updateFastened](#updatefastened) — Update fastened constraint
- [getFastened](#getfastened) — Get fastened constraint info
- [fastenedOrigin](#fastenedorigin) — Create fastened-to-origin constraint
- [updateFastenedOrigin](#updatefastenedorigin) — Update fastened origin constraint
- [getFastenedOrigin](#getfastenedorigin) — Get fastened origin constraint info

### Revolute Constraints
- [revolute](#revolute) — Create revolute constraint (rotation around Z)
- [updateRevolute](#updaterevolute) — Update revolute constraint
- [getRevolute](#getrevolute) — Get revolute constraint info

### Cylindrical Constraints
- [cylindrical](#cylindrical) — Create cylindrical constraint (rotation + slide along Z)
- [updateCylindrical](#updatecylindrical) — Update cylindrical constraint
- [getCylindrical](#getcylindrical) — Get cylindrical constraint info

### Parallel Constraints
- [parallel](#parallel) — Create parallel constraint (Z-axes aligned)
- [updateParallel](#updateparallel) — Update parallel constraint
- [getParallel](#getparallel) — Get parallel constraint info

### Planar Constraints
- [planar](#planar) — Create planar constraint
- [updatePlanar](#updateplanar) — Update planar constraint
- [getPlanar](#getplanar) — Get planar constraint info

### Slider Constraints
- [slider](#slider) — Create slider constraint (slide along Z)
- [updateSlider](#updateslider) — Update slider constraint
- [getSlider](#getslider) — Get slider constraint info

### Spherical Constraints
- [spherical](#spherical) — Create spherical constraint
- [updateSpherical](#updatespherical) — Update spherical constraint
- [getSpherical](#getspherical) — Get spherical constraint info

### Gear Constraints
- [gear](#gear) — Create gear constraint
- [updateGear](#updategear) — Update gear constraint
- [getGear](#getgear) — Get gear constraint info

### Group Constraints
- [group](#group) — Create group constraint
- [updateGroup](#updategroup) — Update group constraint
- [getGroup](#getgroup) — Get group constraint info

### Patterns
- [linearPattern](#linearpattern) — Create linear pattern constraint
- [updateLinearPattern](#updatelinearpattern) — Update linear pattern
- [getLinearPattern](#getlinearpattern) — Get linear pattern info
- [circularPattern](#circularpattern) — Create circular pattern constraint
- [updateCircularPattern](#updatecircularpattern) — Update circular pattern
- [getCircularPattern](#getcircularpattern) — Get circular pattern info

### Constraint Management
- [update3DConstraintValue](#update3dconstraintvalue) — Update a constraint driven value
- [deleteConstraint](#deleteconstraint) — Delete a constraint

### Transform & Motion
- [transformInstance](#transforminstance) — Transform an instance (relative)
- [transformInstanceTo](#transforminstanceto) — Transform an instance (absolute)
- [startMovingUnderConstraints](#startmovingunderconstraints) — Begin constrained drag
- [moveUnderConstraints](#moveunderconstraints) — Move during constrained drag
- [finishMovingUnderConstraints](#finishmovingunderconstraints) — End constrained drag

### Mass Properties
- [calculateMassProperties](#calculatemassproperties) — Calculate mass properties

---

> **AGENT HINTS**:
> - **Mate objects**: Most assembly constraints use `mate1`/`mate2` objects with `{ path, csys, flip?, reorient? }`.
> - `path`: array of instance IDs forming the path in the assembly tree.
> - `csys`: a work coordinate system ID on the part.
> - `flip`: `"X"|"-X"|"Y"|"-Y"|"Z"|"-Z"` — default `"Z"`.
> - `reorient`: `"0"|"90"|"180"|"270"` — default `"0"`.
> - **Transformations**: `instance()` accepts either `[origin, xDir, yDir]` or a 4×4 matrix. Set `isLocal: true` for transforms relative to the owner.

---
<a name="partTemplate"></a>

## partTemplate([param])

Creates a new part and adds it as template to the product container.
This part can be used for assembly building.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the new part template
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                | Default                       | Description                          |
| ------------ | ------------------- | ----------------------------- | ------------------------------------ |
| [param]      | <code>object</code> |                               | object containing all the parameters |
| [param.name] | <code>string</code> | <code>&quot;Part&quot;</code> | Name of the new part template        |

**Example**

```js
api.v1.assembly.partTemplate()
api.v1.assembly.partTemplate({ name: 'Part_152' })
```

> **AGENT NOTE (trained 2026-03-19):** Returns numeric id. Default name is "Part". Requires assembly.create() first — without it returns null + "Assembly building is not initialized!" Can create multiple part templates per session. IDs are unique, incrementing (gaps due to internal objects).

<a name="instance"></a>

## instance(param)

Creates instances of products and adds them to root assembly, assembly instance or assembly template.

- If the owner is an instance in the expanded tree, its template in the assembly container will also be updated.
- If the owner is an assembly template, all instances of the template will also be updated

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                           | Default                                | Description                                                                                                                                                                                                                                                                                                                           |
| ---------------------- | ------------------------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                  | <code>object</code> \| <code>Array&lt;object&gt;</code>                        |                                        | object or objects containing the parameters                                                                                                                                                                                                                                                                                           |
| param.productId        | <code>string</code> \| <code>real</code> \| <code>id</code>                    |                                        | id or identifier of the product to instantiate                                                                                                                                                                                                                                                                                        |
| param.ownerId          | <code>string</code> \| <code>real</code> \| <code>id</code>                    |                                        | id or identifier of the owner of to adding instance, owner can be root assembly, assembly instance or assembly template.                                                                                                                                                                                                              |
| [param.transformation] | <code>Array&lt;point&gt;</code> \| <code>Array&lt;Array&lt;real&gt;&gt;</code> | <code>[[0,0,0],[1,0,0],[0,1,0]]</code> | transformation [origin, x-Dir, y-Dir] of the instance in global coordinates (default=[[0,0,0],[1,0,0],[0,1,0]]) or a 4x4 matrix e.g. [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]] Important: - Left-handed transformation matrices are not yet supported - Scaling part of the 4x4 matrix will be ignored. - Matrices must be orthogonal |
| [param.name]           | <code>string</code>                                                            |                                        | name for the instance, if not set, the instance will get the name of the product with additional unique number, e.g. "MyPart0", "MyPart1"                                                                                                                                                                                             |
| [param.ident]          | <code>string</code>                                                            |                                        | custom string identifier for the added instance.                                                                                                                                                                                                                                                                                      |
| [param.isLocal]        | <code>boolean</code>                                                           | <code>FALSE</code>                     | flag to define whether the transformation is local to owner or global (default=FALSE). This flag gets important if instances are created and added in sub assemblies. In this case, you probably want to add the instance locally to its owner.                                                                                       |

**Example**

```js
api.v1.assembly.instance({ productId: part, ownerId: assemblyRoot })
api.v1.assembly.instance({
  productId: assembly,
  ownerId: instance,
  transformation: [
    [125, 0, 250],
    [1, 0, 0],
    [0, 1, 0],
  ],
})
api.v1.assembly.instance({
  productId: part,
  ownerId: template,
  transformation: [
    [1, 0, 0, 125],
    [0, 1, 0, 0],
    [0, 0, 1, 250],
    [0, 0, 0, 1],
  ],
  isLocal: TRUE,
})
```

> **AGENT NOTE (trained 2026-03-19):** Returns single numeric id (e.g. 112). Batch mode (array of params) creates all instances but result is only the FIRST id, not an array. Both transformation formats work: `[[origin],[xDir],[yDir]]` (3-point) and 4x4 matrix `[[r00,r01,r02,tx],[r10,r11,r12,ty],[r20,r21,r22,tz],[0,0,0,1]]`. `isLocal: true` is critical for sub-assemblies — makes transformation relative to owner's coordinate system. Works with ownerId = root assembly, assembly template, or instance.

<a name="create"></a>

## create([param])

Creates a new root assembly. This is the top level assembly.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created root assembly
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param         | Type                | Default                               | Description                                            |
| ------------- | ------------------- | ------------------------------------- | ------------------------------------------------------ |
| [param]       | <code>object</code> |                                       | object containing all the parameters                   |
| [param.name]  | <code>string</code> | <code>&quot;AssemblyRoot&quot;</code> | name for the root assembly                             |
| [param.ident] | <code>string</code> |                                       | custom string identifier for the created root assembly |

**Example**

```js
api.v1.assembly.create()
```

> **AGENT NOTE (trained 2026-03-19):** Returns numeric id (e.g. 12). Only ONE root assembly allowed per session — calling create() a second time returns null with error "There is already a root assembly or part which must be removed first." Must be called before partTemplate()/assemblyTemplate() or those will fail with "Assembly building is not initialized!"

<a name="getPartTemplate"></a>

## getPartTemplate([param])

Returns the part template with given name from part container.
If name is empty, all parts from part container will be returned.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|Array<id>  // id or ids of the found part templates
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                | Description                          |
| ------------ | ------------------- | ------------------------------------ |
| [param]      | <code>object</code> | object containing all the parameters |
| [param.name] | <code>string</code> | name of the part to look for         |

**Example**

```js
api.v1.assembly.getPartTemplate()
api.v1.assembly.getPartTemplate({ name: 'Part_125' })
```

> **AGENT NOTE (trained 2026-03-19):** With `{name:'X'}` returns single id or null + error if not found. With no params `[]` returns array of ALL part template ids. Non-existent name → null + "Part with name = "X" could not be found".

<a name="linearPattern"></a>

## linearPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{constraint: id, instances: [id, ...]}`. `count=3` creates 3 new instances (4 total incl. original). Default dir2 is `{count:1, distance:100}` (unused). dir1 follows mate1.csys flip axis (Z default). Flip X/Y/Z changes pattern direction. Negative distances work. 2D grid requires mate2 for dir2 to take effect — without mate2, dir2 is ignored. Batch array param works but returns only the last result.

Creates a new linear pattern constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { constraint: id, instances: Array<id> }|VOID|Array<{ constraint: id, instances: Array<id> }|VOID>  // Object or objects with ids of the created linear pattern constraints and their instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                                                                                                                                | Default                                | Description                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------- |
| param                  | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                        | object or objects containing the parameters                                     |
| param.id               | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                        | id of the assembly to create the linear pattern constraint in                   |
| [param.name]           | <code>string</code>                                                                                                                                                                 | <code>&quot;LinearPattern&quot;</code> | name for the linear pattern constraint                                          |
| param.instanceId       | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                        | id of the instance to be copied                                                 |
| param.mate1            | <code>object</code>                                                                                                                                                                 |                                        | object containing the parameters for mate 1                                     |
| param.mate1.path       | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| param.mate1.csys       | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                        | id of the work coordinate system to use for this first mate                     |
| [param.mate1.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>             | flip type which defines the global main axis of the mate (default="Z")          |
| [param.mate1.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>             | orientation around the main axis in 90° steps (default="0")                     |
| [param.dir1]           | <code>object</code>                                                                                                                                                                 |                                        | object containing the parameters for the first direction of the linear pattern  |
| [param.dir1.count]     | <code>real</code>                                                                                                                                                                   | <code>1</code>                         | number of copies over the first direction (default=1)                           |
| [param.dir1.distance]  | <code>real</code>                                                                                                                                                                   | <code>100</code>                       | distance between copies over the first direction (default=100)                  |
| [param.mate2]          | <code>object</code>                                                                                                                                                                 |                                        | object containing the parameters for mate 2                                     |
| param.mate2.path       | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| param.mate2.csys       | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                        | id of the work coordinate system to use for this second mate                    |
| [param.mate2.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>             | flip type which defines the global main axis of the mate (default="Z")          |
| [param.mate2.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>             | orientation around the main axis in 90° steps (default="0")                     |
| [param.dir2]           | <code>object</code>                                                                                                                                                                 |                                        | object containing the parameters for the second direction of the linear pattern |
| [param.dir2.count]     | <code>real</code>                                                                                                                                                                   | <code>1</code>                         | number of copies over the second direction (default=1)                          |
| [param.dir2.distance]  | <code>real</code>                                                                                                                                                                   | <code>100</code>                       | distance between copies over the second direction (default=100)                 |

**Example**

```js
api.v1.assembly.linearPattern({ id: assembly, instanceId: instance4, mate1: { path: [instance4], csys: wcs } })
```

<a name="updateLinearPattern"></a>

## updateLinearPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Takes `{id: constraintId, ...}` — id is the constraint id from linearPattern result, NOT the assembly id. Can update dir1/dir2 count+distance and mate1 (including flip). Returns `{constraint, instances}` with updated instance list.

Updates an existing linear pattern constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { constraint: id, instances: Array<id> }|VOID|Array<{ constraint: id, instances: Array<id> }|VOID>  // Object or objects with ids of the created linear pattern constraints and their instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                                                                                                                                | Description                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| param                  | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                     |
| param.id               | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the linear pattern constraint to update                                   |
| [param.name]           | <code>string</code>                                                                                                                                                                 | name for the linear pattern constraint                                          |
| [param.instanceId]     | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the instance to be copied                                                 |
| [param.mate1]          | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                     |
| [param.mate1.path]     | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| [param.mate1.csys]     | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                     |
| [param.mate1.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                        |
| [param.mate1.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                   |
| [param.dir1]           | <code>object</code>                                                                                                                                                                 | object containing the parameters for the first direction of the linear pattern  |
| [param.dir1.count]     | <code>real</code>                                                                                                                                                                   | number of copies over the first direction                                       |
| [param.dir1.distance]  | <code>real</code>                                                                                                                                                                   | distance between copies over the first direction                                |
| [param.mate2]          | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                     |
| [param.mate2.path]     | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| [param.mate2.csys]     | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                    |
| [param.mate2.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                        |
| [param.mate2.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                   |
| [param.dir2]           | <code>object</code>                                                                                                                                                                 | object containing the parameters for the second direction of the linear pattern |
| [param.dir2.count]     | <code>real</code>                                                                                                                                                                   | number of copies over the second direction                                      |
| [param.dir2.distance]  | <code>real</code>                                                                                                                                                                   | distance between copies over the second direction                               |

**Example**

```js
api.v1.assembly.updateLinearPattern({ id: pattern, instanceId: instance, dir1: { count: 4, distance: 150 } })
```

<a name="getLinearPattern"></a>

## getLinearPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, instanceId, name, mate1:{csys, flip, path, reorient}, dir1:{count, distance}, dir2:{count, distance}}`. Query by assembly/instance id + constraint name. Default flip is "Z", default reorient is "0".

Returns the linear pattern constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    instanceId: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2?: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     dir1: {
       count: real,
       distance: real
     },
     dir2: {
       count: real,
       distance: real
     }
   } | Array<{
     id: id
    instanceId: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2?: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     dir1: {
       count: real,
       distance: real
     },
     dir2: {
       count: real,
       distance: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getLinearPattern({ id: assemblyTemplate, name: 'LP1' })
```

<a name="getAssemblyTemplate"></a>

## getAssemblyTemplate([param])

Returns the assembly template with given name from assembly container.
If name is empty, all assembly from assembly container will be returned.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|Array<id>  // id or ids of the found assembly templates
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                | Description                          |
| ------------ | ------------------- | ------------------------------------ |
| [param]      | <code>object</code> | object containing all the parameters |
| [param.name] | <code>string</code> | name of the assembly to look for     |

**Example**

```js
api.v1.assembly.getAssemblyTemplate()
api.v1.assembly.getAssemblyTemplate({ name: 'Assembly_02' })
```

> **AGENT NOTE (trained 2026-03-19):** Same pattern as getPartTemplate. With `{name:'X'}` returns single id or null. With no params returns array of all assembly template ids.

<a name="getInstance"></a>

## getInstance(param)

Returns a single, mutliple or all instances of an owner, depending on whether a name is given or not.
The owner is the parent of the instance/instances we're looking for and can be a root assembly,
another instance or an assembly template.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the found instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param         | Type                                                        | Description                                                                                                   |
| ------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| param         | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing all the parameters                                                               |
| param.ownerId | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the owner from which we are searching the children for instances that have the name we are looking for. |
| [param.name]  | <code>string</code>                                         | name of the instance to look for                                                                              |

**Example**

```js
api.v1.assembly.getInstance({ ownerId: assemblyRoot, name: 'Inst0' }) // return value = 10
api.v1.assembly.getInstance([
  { ownerId: instance, name: 'Inst0' },
  { ownerId: instance2, name: 'Inst1' },
]) // return value = [10, 15]
api.v1.assembly.getInstance({ ownerId: assemblyTemplate }) // return value = [10, 15, 20]
```

> **AGENT NOTE (trained 2026-03-19):** By name → returns single numeric id. No name → returns array of ALL child instance ids. Non-existent name → returns empty array `[]` (no error, no messages).

<a name="deleteInstance"></a>

## deleteInstance(param)

Deletes instances from root assembly, other instances or assembly templates.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param     | Type                                         | Description                                   |
| --------- | -------------------------------------------- | --------------------------------------------- |
| param     | <code>object</code>                          | object containing all the parameters          |
| param.ids | <code>Array&lt;(string\|real\|id)&gt;</code> | ids or identifiers of the instances to delete |

**Example**

```js
api.v1.assembly.deleteInstance({ ids: [instance1, instance2, instance3] })
```

> **AGENT NOTE (trained 2026-03-19):** Takes `{ ids: [...] }` — array of ids. Multiple ids in one call works. Returns null. Template is NOT deleted, only the instances.

<a name="fastened"></a>

## fastened(param)

Creates a new fastened constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

> **AGENT NOTE (trained 2026-03-19):** Returns constraint ID (integer). Creates `CC_FastenedConstraint` under ConstraintSet. `mate.csys` requires a `CC_WorkCSys` ID created via `v1.part.workCSys` — `getWorkGeometry` does NOT work on part templates. `mate.path` is `[instanceId]` for single-level assemblies. Offsets map directly to instance coordinateSystem translation (e.g. xOffset=80, zOffset=50 → origin at [80,0,50]). Batch creation works: pass array of constraint objects. `useCurrentTransform: true` stored offsets as [0,0,0] even with pre-positioned instance — may capture relative transform between mates, not absolute position. **Rotation params** (zRotation with `'45deg'` string or radians number) stored as 0 in constraint members — needs further investigation.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created fastened constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default                           | Description                                                                                                                                                         |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                   | object or objects containing the parameters                                                                                                                         |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the assembly to create the fastened constraint in                                                                                                             |
| [param.name]                | <code>string</code>                                                                                                                                                                 | <code>&quot;Fastened&quot;</code> | name for the fastened constraint (default="Fastened")                                                                                                               |
| param.mate1                 | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 1                                                                                                                         |
| param.mate1.path            | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                                     |
| param.mate1.csys            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this first mate                                                                                                         |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate (default="Z")                                                                                              |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                                         |
| param.mate2                 | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 2                                                                                                                         |
| param.mate2.path            | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                                     |
| param.mate2.csys            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this second mate                                                                                                        |
| [param.mate2.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate 2 (default="Z")                                                                                            |
| [param.mate2.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                                         |
| [param.xOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                    | offset along x-axis from mate1 to mate2 (default=0)                                                                                                                 |
| [param.yOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                    | offset along y-axis from mate1 to mate2 (default=0)                                                                                                                 |
| [param.zOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                    | offset along z-axis from mate1 to mate2 (default=0)                                                                                                                 |
| [param.xRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                    | rotation of mate2 in radians around x-axis of mate1 (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.yRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                    | rotation of mate2 in radians around y-axis of mate1 (default=0) It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ...  |
| [param.zRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                    | rotation of mate2 in radians around z-axis of mate1 (default=0) It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ...  |
| [param.useCurrentTransform] | <code>boolean</code>                                                                                                                                                                | <code>FALSE</code>                | a flag to ignore specified offsets / rotations and recalculate them according to mates' current transforms (default=FALSE)                                          |

**Example**

```js
api.v1.assembly.fastened({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance3], csys: wcs } })
api.v1.assembly.fastened({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance3], csys: wcs }, xRotation: '45deg' })
```

<a name="updateFastened"></a>

## updateFastened(param)

Updates an existing fastened constraint.
If optional parameters are not set, the constraint will keep the existing values.

> **AGENT NOTE (trained 2026-03-19):** Takes the constraint ID (returned from `fastened`), NOT assembly ID + name. Partial updates work: can change just offsets without re-specifying mates. Rotation update with `'90deg'` string stored zRotation as 0 — same rotation issue as creation.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated fastened constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default            | Description                                                                                                                                             |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                    | object or objects containing the parameters                                                                                                             |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                    | id of the fastened constraint to update                                                                                                                 |
| [param.name]                | <code>string</code>                                                                                                                                                                 |                    | name for the fastened constraint                                                                                                                        |
| [param.mate1]               | <code>object</code>                                                                                                                                                                 |                    | object containing the parameters for mate 1                                                                                                             |
| [param.mate1.path]          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                    | an array with a full mate path (or with a single instance in the expanded tree)                                                                         |
| [param.mate1.csys]          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                    | id of the work coordinate system to use for this first mate                                                                                             |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> |                    | flip type which defines the global main axis of the mate                                                                                                |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           |                    | orientation around the main axis in 90° steps                                                                                                           |
| [param.mate2]               | <code>object</code>                                                                                                                                                                 |                    | object containing the parameters for mate 2                                                                                                             |
| [param.mate2.path]          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                    | an array with a full mate path (or with a single instance in the expanded tree)                                                                         |
| [param.mate2.csys]          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                    | id of the work coordinate system to use for this second mate                                                                                            |
| [param.mate2.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> |                    | flip type which defines the global main axis of the mate 2                                                                                              |
| [param.mate2.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           |                    | orientation around the main axis in 90° steps                                                                                                           |
| [param.xOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along x-axis from mate1 to mate2                                                                                                                 |
| [param.yOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along y-axis from mate1 to mate2                                                                                                                 |
| [param.zOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along z-axis from mate1 to mate2                                                                                                                 |
| [param.xRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around x-axis of mate1. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.yRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around y-axis of mate1. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.zRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around z-axis of mate1. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.useCurrentTransform] | <code>boolean</code>                                                                                                                                                                | <code>FALSE</code> | a flag to ignore specified offsets / rotations and recalculate them according to mates' current transforms (default=FALSE)                              |

**Example**

```js
api.v1.assembly.updateFastened({ id: constraint, mate1: { csys: wcs1 }, xOffset: 150 })
api.v1.assembly.updateFastened({ id: constraint, xRotation: '45deg', yRotation: 1.57 })
```

<a name="getFastened"></a>

## getFastened(param)

Returns the fastened constraint of given reference with specified name.

> **AGENT NOTE (trained 2026-03-19):** Takes assembly ID + constraint name. Returns full constraint data including offsets, rotations, and mate references. Non-existent name returns `null` with level 51 ERROR: "There couldn't be found a constraint with name X".

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
 {
  result: {
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    xOffset: real
    yOffset: real
    zOffset: real
    xRotation: real
    yRotation: real
    zRotation: real
  } | Array<{
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    xOffset: real
    yOffset: real
    zOffset: real
    xRotation: real
    yRotation: real
    zRotation: real
  } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                           |
| ---------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters           |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the assembly or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for                |

**Example**

```js
api.v1.assembly.getFastened({ id: assembly, name: 'Fastened3' })
```

<a name="getFastenedOrigin"></a>

## getFastenedOrigin(param)

Returns the fastened origin constraint of given reference with specified name.

> **AGENT NOTE (trained 2026-03-19):** Same pattern as `getFastened` but returns `CC_FastenedOriginConstraint` data. Takes assembly ID + name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
 {
  result: {
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    xOffset: real
    yOffset: real
    zOffset: real
    xRotation: real
    yRotation: real
    zRotation: real
  } | Array<{
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    xOffset: real
    yOffset: real
    zOffset: real
    xRotation: real
    yRotation: real
    zRotation: real
  } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getFastenedOrigin({ id: assembly, name: 'FO_5' })
```

<a name="fastenedOrigin"></a>

## fastenedOrigin(param)

Creates a new fastened origin constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

> **AGENT NOTE (trained 2026-03-19):** Creates `CC_FastenedOriginConstraint` (distinct class from `CC_FastenedConstraint`). Only takes `mate1` — anchors instance to assembly global origin. Offsets and flip/reorient accepted same as `fastened`. `useCurrentTransform` stored offsets as [0,0,0] despite pre-positioned instance.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created fastened origin constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default                                 | Description                                                                                                                                                                |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                         | object or objects containing the parameters                                                                                                                                |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                         | id of the assembly to create the fastened origin constraint in                                                                                                             |
| [param.name]                | <code>string</code>                                                                                                                                                                 | <code>&quot;FastenedOrigin&quot;</code> | name for the fastened origin constraint (default="FastenedOrigin")                                                                                                         |
| param.mate1                 | <code>object</code>                                                                                                                                                                 |                                         | object containing the parameters for mate 1                                                                                                                                |
| param.mate1.path            | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                         | an array with a full mate path (or with a single instance in the expanded tree)                                                                                            |
| param.mate1.csys            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                         | id of the work coordinate system to use for this first mate                                                                                                                |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>              | flip type which defines the global main axis of the mate (default="Z")                                                                                                     |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>              | orientation around the main axis in 90° steps (default="0")                                                                                                                |
| [param.xOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                          | offset along x-axis from mate1 to global origin (default=0)                                                                                                                |
| [param.yOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                          | offset along y-axis from mate1 to global origin (default=0)                                                                                                                |
| [param.zOffset]             | <code>real</code>                                                                                                                                                                   | <code>0</code>                          | offset along z-axis from mate1 to global origin (default=0)                                                                                                                |
| [param.xRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                          | rotation of mate2 in radians around x-axis of global origin (default=0) It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.yRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                          | rotation of mate2 in radians around y-axis of global origin (default=0) It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.zRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                          | rotation of mate2 in radians around z-axis of global origin (default=0) It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.useCurrentTransform] | <code>real</code>                                                                                                                                                                   | <code>FALSE</code>                      | a flag to ignore specified offsets / rotations and recalculate them according to mates' current transforms (default=FALSE)                                                 |

**Example**

```js
api.v1.assembly.fastenedOrigin({ id: assembly, mate1: { path: [instance], csys: wcs, flip: '-X' }, xRotation: 3.14 })
```

<a name="updateFastenedOrigin"></a>

## updateFastenedOrigin(param)

Updates an existing fastened origin constraint.
If optional parameters are not set, the constraint will keep the existing values.

> **AGENT NOTE (trained 2026-03-19):** Same pattern as `updateFastened` — takes constraint ID, supports partial updates of offsets/rotations.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created fastened origin constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default            | Description                                                                                                                                                                 |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                    | object or objects containing the parameters                                                                                                                                 |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                    | id of the fastened origin constraint to update                                                                                                                              |
| [param.name]                | <code>string</code>                                                                                                                                                                 |                    | name for the fastened origin constraint                                                                                                                                     |
| [param.mate1]               | <code>object</code>                                                                                                                                                                 |                    | object containing the parameters for mate 1                                                                                                                                 |
| [param.mate1.path]          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                    | an array with a full mate path (or with a single instance in the expanded tree)                                                                                             |
| [param.mate1.csys]          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                    | id of the work coordinate system to use for this first mate                                                                                                                 |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> |                    | flip type which defines the global main axis of the mate                                                                                                                    |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           |                    | orientation around the main axis in 90° steps                                                                                                                               |
| [param.xOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along x-axis from mate1 to global origin (default=0)                                                                                                                 |
| [param.yOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along y-axis from mate1 to global origin (default=0)                                                                                                                 |
| [param.zOffset]             | <code>real</code>                                                                                                                                                                   |                    | offset along z-axis from mate1 to global origin (default=0)                                                                                                                 |
| [param.xRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around x-axis of global origin (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.yRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around y-axis of global origin (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.zRotation]           | <code>real</code> \| <code>expression</code>                                                                                                                                        |                    | rotation of mate2 in radians around z-axis of global origin (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.useCurrentTransform] | <code>boolean</code>                                                                                                                                                                | <code>FALSE</code> | a flag to ignore specified offsets / rotations and recalculate them according to mates' current transforms (default=FALSE)                                                  |

**Example**

```js
api.v1.assembly.updateFastenedOrigin({ id: constraint, xOffset: 150, zOffset: 58 })
api.v1.assembly.updateFastenedOrigin({ id: constraint, xOffset: 150, zOffset: 58, zRotation: '15.5deg' })
```

<a name="revolute"></a>

## revolute(param)

> **AGENT NOTE (trained 2026-03-19):** Returns a numeric constraint ID. 1-DOF rotation around the main axis (default Z). `zOffset` shifts mate2 along the rotation axis. `zRotationLimits` accepts degree expressions like `'-90deg'`, `'270deg'` — stored internally as radians. `flip` changes which WCS axis becomes the rotation axis (e.g. `flip: 'X'` rotates around X). Batch array params return a single ID (appears to be the last), NOT an array — do NOT rely on batch for multiple constraints, create them one at a time.

Creates a new revolute constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created revolute constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Default                           | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                   | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the assembly to create the revolute constraint in                                                                                            |
| [param.name]              | <code>string</code>                                                                                                                                                                 | <code>&quot;Revolute&quot;</code> | name for the revolute constraint (default="Revolute")                                                                                              |
| param.mate1               | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 1                                                                                                        |
| param.mate1.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate1.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate (default="Z")                                                                             |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                        |
| param.mate2               | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 2                                                                                                        |
| param.mate2.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate2.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate 2 (default="Z")                                                                           |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                        |
| [param.zOffset]           | <code>real</code>                                                                                                                                                                   | <code>0</code>                    | offset along z-axis from mate1 to mate2 (default=0)                                                                                                |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                   | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                   | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                   | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.revolute({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs2 } })
```

<a name="updateRevolute"></a>

## updateRevolute(param)

> **AGENT NOTE (trained 2026-03-19):** Supports partial updates — can change just `zOffset` or `zRotationLimits` without re-specifying mates. Degree expressions work in limits. Returns the same constraint ID on success.

Updates a revolute constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated revolute constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the revolute constraint to update                                                                                                            |
| [param.name]              | <code>string</code>                                                                                                                                                                 | name for the revolute constraint                                                                                                                   |
| [param.mate1]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                                                                                        |
| [param.mate1.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate1.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                                                                                           |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.mate2]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                                                                                        |
| [param.mate2.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate2.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate 2                                                                                         |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.zOffset]           | <code>real</code>                                                                                                                                                                   | offset along z-axis from mate1 to mate2                                                                                                            |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updateRevolute({ id: constraint, mate2: { flip: '-Z' } })
api.v1.assembly.updateRevolute({ id: constraint, zRotationLimits: { min: '-180deg', max: '270deg' } })
```

<a name="getRevolute"></a>

## getRevolute(param)

> **AGENT NOTE (trained 2026-03-19):** Takes `{ id: assemblyId, name: constraintName }`. Returns full constraint state including mate paths, csys, flip, reorient, zOffset, and zRotationLimits (in radians). Returns `null` with ERROR message if name not found.

Returns the revolute constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
 {
  result: {
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    zOffset: real
    zRotationLimits: {
       min: real, max: real
     }
  } | Array<{
    id: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
    }
    zOffset: real
    zRotationLimits: {
       min: real, max: real
     }
  } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getRevolute({ id: assembly, name: 'Rev_12' })
```

<a name="circularPattern"></a>

## circularPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{constraint: id, instances: [id, ...]}`. `instanceCount=4` creates 4 new instances (5 total incl. original). Angle accepts both "90deg" string and raw radians — always stored/returned as radians. Negative angles reverse rotation direction. `offset` = Z-axis displacement per copy (helical pattern). Rotation axis = mate1.csys flip axis (Z by default).

Creates a new circular pattern constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { constraint: id, instances: Array<id> }|VOID|Array<{ constraint: id, instances: Array<id> }|VOID>  // Object or objects with ids of the created circular pattern constraints and their instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                                                                                                                                | Default                                  | Description                                                                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| param                  | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                          | object or objects containing the parameters                                                                                                     |
| param.id               | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                          | id of the assembly to create the circular pattern constraint in                                                                                 |
| [param.name]           | <code>string</code>                                                                                                                                                                 | <code>&quot;CircularPattern&quot;</code> | name for the circular pattern constraint (default="CircularPattern")                                                                            |
| param.instanceId       | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                          | id of the instance to be copied                                                                                                                 |
| param.mate1            | <code>object</code>                                                                                                                                                                 |                                          | object containing the parameters for this mate                                                                                                  |
| param.mate1.path       | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                          | an array with a full mate path (or with a single instance in the expanded tree)                                                                 |
| param.mate1.csys       | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                          | id of the work coordinate system to use for this mate                                                                                           |
| [param.mate1.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>               | flip type which defines the global main axis of the mate (default="Z")                                                                          |
| [param.mate1.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>               | orientation around the main axis in 90° steps (default="0")                                                                                     |
| [param.instanceCount]  | <code>real</code>                                                                                                                                                                   | <code>1</code>                           | number of copies (default=1)                                                                                                                    |
| [param.angle]          | <code>real</code> \| <code>expression</code>                                                                                                                                        | <code>0</code>                           | angle in radians between copies (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.offset]         | <code>real</code>                                                                                                                                                                   | <code>0</code>                           | offset over the rotational axis between copies (default=0)                                                                                      |

**Example**

```js
api.v1.assembly.circularPattern({ id: assembly, instanceId: instance, mate1: { path: [instance], csys: wcs }, instanceCount: 4, angle: 1.57 })
api.v1.assembly.circularPattern({ id: assembly, instanceId: instance, mate1: { path: [instance], csys: wcs }, instanceCount: 4, angle: '90deg' })
```

<a name="updateCircularPattern"></a>

## updateCircularPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Takes `{id: constraintId, ...}`. Can update instanceCount, angle, offset independently. Changing instanceCount grows/shrinks instances array. Returns `{constraint, instances}`.

Updates an existing circular pattern constraint.
If optional parameters are not set, the feature will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { constraint: id, instances: Array<id> }|VOID|Array<{ constraint: id, instances: Array<id> }|VOID>  // Object or objects with ids of the created circular pattern constraints and their instances
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                  | Type                                                                                                                                                                                | Description                                                                                                                         |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| param                  | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                                                                         |
| param.id               | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the circular pattern constraint to update                                                                                     |
| [param.name]           | <code>string</code>                                                                                                                                                                 | name for the circular pattern constraint                                                                                            |
| [param.instanceId]     | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the instance to be copied                                                                                                     |
| [param.mate1]          | <code>object</code>                                                                                                                                                                 | object containing the parameters for this mate                                                                                      |
| [param.mate1.path]     | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                     |
| [param.mate1.csys]     | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this mate                                                                               |
| [param.mate1.flip]     | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                                                                            |
| [param.mate1.reorient] | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                       |
| [param.instanceCount]  | <code>real</code>                                                                                                                                                                   | number of copies                                                                                                                    |
| [param.angle]          | <code>real</code> \| <code>expression</code>                                                                                                                                        | angle in radians between copies. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.offset]         | <code>real</code>                                                                                                                                                                   | offset over the rotational axis between copies                                                                                      |

**Example**

```js
api.v1.assembly.updateCircularPattern({ id: constraint, instanceCount: 3, angle: 2.09 })
```

<a name="cylindrical"></a>

## cylindrical(param)

> **AGENT NOTE (trained 2026-03-19):** 2-DOF constraint: rotation + translation along Z axis. Has both `zOffsetLimits` (translation range) and `zRotationLimits` (rotation range) — can be set simultaneously. Degree expressions work. Same mate structure as revolute.

Creates a new cylindrical constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created cylindrical constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Default                              | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                      | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                      | id of the assembly to create the cylindrical constraint in                                                                                         |
| [param.name]              | <code>string</code>                                                                                                                                                                 | <code>&quot;Cylindrical&quot;</code> | name for the cylindrical constraint (default="Cylindrical")                                                                                        |
| param.mate1               | <code>object</code>                                                                                                                                                                 |                                      | object containing the parameters for mate 1                                                                                                        |
| param.mate1.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                      | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate1.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                      | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>           | flip type which defines the global main axis of the mate (default="Z")                                                                             |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>           | orientation around the main axis in 90° steps (default="0")                                                                                        |
| param.mate2               | <code>object</code>                                                                                                                                                                 |                                      | object containing the parameters for mate 2                                                                                                        |
| param.mate2.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                      | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate2.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                      | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>           | flip type which defines the global main axis of the mate 2 (default="Z")                                                                           |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>           | orientation around the main axis in 90° steps (default="0")                                                                                        |
| [param.zOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                      | object defining min and max value of the offset around z-axis                                                                                      |
| param.zOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                      | min value of the offset along z-axis                                                                                                               |
| param.zOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                      | max value value of the offset along z-axis                                                                                                         |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                      | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                      | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                      | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.cylindrical({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs2 } })
```

<a name="updateCylindrical"></a>

## updateCylindrical(param)

> **AGENT NOTE (trained 2026-03-19):** Supports partial updates. Passing `null` for limits clears them (restores unlimited motion). Can set both `zOffsetLimits` and `zRotationLimits` in one call.

Updates an existing cylindrical constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated cylindrical constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the cylindrical constraint to update                                                                                                         |
| [param.name]              | <code>string</code>                                                                                                                                                                 | name for the cylindrical constraint                                                                                                                |
| [param.mate1]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                                                                                        |
| [param.mate1.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate1.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                                                                                           |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.mate2]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                                                                                        |
| [param.mate2.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate2.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate 2                                                                                         |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.zOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of the offset around z-axis                                                                                      |
| param.zOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | min value of the offset along z-axis                                                                                                               |
| param.zOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | max value value of the offset along z-axis                                                                                                         |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updateCylindrical({ id: constraint, zRotationLimits: { min: 0, max: 1.57 } })
```

<a name="parallel"></a>

## parallel(param)

> **AGENT NOTE (trained 2026-03-19):** 5-DOF constraint: aligns Z axes, leaves 3 translations + Z rotation free. Has `xOffsetLimits`, `yOffsetLimits`, `zOffsetLimits`, and `zRotationLimits`. Set `zOffsetLimits: { min: 0, max: 0 }` to lock Z translation for planar sliding. All limits support partial updates.

Creates a new parallel constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created parallel constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default                           | Description                                                                                                                                        |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                   | object or objects containing the parameters                                                                                                        |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the assembly to create the parallel constraint in                                                                                            |
| [param.name]                | <code>string</code>                                                                                                                                                                 | <code>&quot;Parallel&quot;</code> | name for the parallel constraint (default="Parallel")                                                                                              |
| param.mate1                 | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 1                                                                                                        |
| param.mate1.path            | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate1.csys            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate (default="Z")                                                                             |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                        |
| param.mate2                 | <code>object</code>                                                                                                                                                                 |                                   | object containing the parameters for mate 2                                                                                                        |
| param.mate2.path            | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                   | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate2.csys            | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                   | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>        | flip type which defines the global main axis of the mate 2 (default="Z")                                                                           |
| [param.mate2.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>        | orientation around the main axis in 90° steps (default="0")                                                                                        |
| [param.xOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                   | object defining min and max value of the offset around x-axis                                                                                      |
| [param.xOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | min value of the offset along x-axis                                                                                                               |
| [param.xOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | max value value of the offset along x-axis                                                                                                         |
| [param.yOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                   | object defining min and max value of the offset around y-axis                                                                                      |
| [param.yOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | min value of the offset along y-axis                                                                                                               |
| [param.yOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | max value value of the offset along y-axis                                                                                                         |
| [param.zOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                   | object defining min and max value of the offset around z-axis                                                                                      |
| [param.zOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | min value of the offset along z-axis                                                                                                               |
| [param.zOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                   | max value value of the offset along z-axis                                                                                                         |
| [param.zRotationLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                   | object defining min and max value of rotation around z-axis                                                                                        |
| [param.zRotationLimits.min] | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                   | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.zRotationLimits.max] | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                   | max value of rotation around z-axis in radians It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ...  |

**Example**

```js
api.v1.assembly.parallel({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs } })
```

<a name="updateParallel"></a>

## updateParallel(param)

> **AGENT NOTE (trained 2026-03-19):** Supports partial updates — can add/change individual limit sets (x/y/z offset, zRotation) without touching others. Degree expressions work in zRotationLimits.

Updates an existing parallel constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated parallel constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                       | Type                                                                                                                                                                                | Default                    | Description                                                                                                                                        |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                            | object or objects containing the parameters                                                                                                        |
| param.id                    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                            | id of the parallel constraint to update                                                                                                            |
| [param.name]                | <code>string</code>                                                                                                                                                                 |                            | name for the parallel constraint                                                                                                                   |
| [param.mate1]               | <code>object</code>                                                                                                                                                                 |                            | object containing the parameters for mate 1                                                                                                        |
| [param.mate1.path]          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                            | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate1.csys]          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                            | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> |                            | flip type which defines the global main axis of the mate                                                                                           |
| [param.mate1.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code> | orientation around the main axis in 90° steps                                                                                                      |
| [param.mate2]               | <code>object</code>                                                                                                                                                                 |                            | object containing the parameters for mate 2                                                                                                        |
| [param.mate2.path]          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                            | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate2.csys]          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                            | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]          | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> |                            | flip type which defines the global main axis of the mate 2                                                                                         |
| [param.mate2.reorient]      | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           |                            | orientation around the main axis in 90° steps                                                                                                      |
| [param.xOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                            | object defining min and max value of the offset around x-axis                                                                                      |
| [param.xOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | min value of the offset along x-axis                                                                                                               |
| [param.xOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | max value value of the offset along x-axis                                                                                                         |
| [param.yOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                            | object defining min and max value of the offset around y-axis                                                                                      |
| [param.yOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | min value of the offset along y-axis                                                                                                               |
| [param.yOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | max value value of the offset along y-axis                                                                                                         |
| [param.zOffsetLimits]       | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                            | object defining min and max value of the offset around z-axis                                                                                      |
| [param.zOffsetLimits.min]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | min value of the offset along z-axis                                                                                                               |
| [param.zOffsetLimits.max]   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                            | max value value of the offset along z-axis                                                                                                         |
| [param.zRotationLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                            | object defining min and max value of rotation around z-axis                                                                                        |
| [param.zRotationLimits.min] | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                            | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| [param.zRotationLimits.max] | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                            | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updateParallel({ id: constraint, zRotationLimits: { min: 0, max: 1.57 } })
```

<a name="planar"></a>

## planar(param)

> **AGENT NOTE (trained 2026-03-19):** 3-DOF constraint: slide on XY plane + rotate around Z normal. `zOffset` fixes perpendicular distance (default 0). `xOffsetLimits`/`yOffsetLimits` constrain sliding range (default null = unconstrained). `zRotationLimits` accepts "45deg" string expressions — stored internally as radians. flip Z/-Z on mate2 flips normal direction (face-to-face vs face-away). reorient "90" rotates mate frame. Batch array param returns single id (only first created — needs investigation).

Creates a new planar constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created planar constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Default                         | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                 | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the assembly to create the planar constraint in                                                                                              |
| [param.name]              | <code>string</code>                                                                                                                                                                 | <code>&quot;Planar&quot;</code> | name for the planar constraint (default="Planar")                                                                                                  |
| param.mate1               | <code>object</code>                                                                                                                                                                 |                                 | object containing the parameters for mate 1                                                                                                        |
| param.mate1.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                 | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate1.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>      | flip type which defines the global main axis of the mate (default="Z")                                                                             |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>      | orientation around the main axis in 90° steps (default="0")                                                                                        |
| param.mate2               | <code>object</code>                                                                                                                                                                 |                                 | object containing the parameters for mate 2                                                                                                        |
| param.mate2.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                 | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate2.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>      | flip type which defines the main axis of the mate 2 (default="Z")                                                                                  |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>      | orientation around the main axis in 90° steps (default="0")                                                                                        |
| [param.zOffset]           | <code>real</code>                                                                                                                                                                   | <code>0</code>                  | offset along z-axis from mate1 to mate2 (default=0)                                                                                                |
| [param.xOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                 | object defining min and max value of the offset around x-axis                                                                                      |
| param.xOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | min value of the offset along x-axis                                                                                                               |
| param.xOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | max value value of the offset along x-axis                                                                                                         |
| [param.yOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                 | object defining min and max value of the offset around y-axis                                                                                      |
| param.yOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | min value of the offset along y-axis                                                                                                               |
| param.yOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | max value value of the offset along y-axis                                                                                                         |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                 | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                 | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                 | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.planar({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs } })
```

<a name="updatePlanar"></a>

## updatePlanar(param)

> **AGENT NOTE (trained 2026-03-19):** Works with partial params — can update just limits without mates. Accepts "90deg" string expressions in zRotationLimits. Returns the constraint id. Takes constraint id (not assembly+name).

Updates an existing planar constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated planar constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the planar constraint to update                                                                                                              |
| [param.name]              | <code>string</code>                                                                                                                                                                 | name for the planar constraint                                                                                                                     |
| [param.mate1]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                                                                                        |
| [param.mate1.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate1.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                                                                                           |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.mate2]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                                                                                        |
| [param.mate2.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate2.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the main axis of the mate 2                                                                                                |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.zOffset]           | <code>real</code>                                                                                                                                                                   | offset along z-axis from mate1 to mate2                                                                                                            |
| [param.xOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of the offset around x-axis                                                                                      |
| param.xOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | min value of the offset along x-axis                                                                                                               |
| param.xOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | max value value of the offset along x-axis                                                                                                         |
| [param.yOffsetLimits]     | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of the offset around y-axis                                                                                      |
| param.yOffsetLimits.min   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | min value of the offset along y-axis                                                                                                               |
| param.yOffsetLimits.max   | <code>real</code> \| <code>VOID</code>                                                                                                                                              | max value value of the offset along y-axis                                                                                                         |
| [param.zRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of rotation around z-axis                                                                                        |
| param.zRotationLimits.min | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | min value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |
| param.zRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | max value of rotation around z-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updatePlanar({ id: constraint, xOffsetLimits: { min: 0, max: 150 } })
api.v1.assembly.updatePlanar({ id: constraint, zRotationLimits: { min: '-90deg', max: '90deg' } })
```

<a name="slider"></a>

## slider(param)

> **AGENT NOTE (trained 2026-03-19):** 1-DOF constraint: translate along Z only. `xOffset`/`yOffset` are fixed lateral offsets (not free DOFs, default 0). `zOffsetLimits` constrains slide range (default null = unconstrained). Axes aligned via flip/reorient on mates.

Creates a new slider constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created slider constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                   | Type                                                                                                                                                                                | Default                         | Description                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------- |
| param                   | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                 | object or objects containing the parameters                                     |
| param.id                | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the assembly to create the slider constraint in                           |
| [param.name]            | <code>string</code>                                                                                                                                                                 | <code>&quot;Slider&quot;</code> | name for the slider constraint (default="Slider")                               |
| param.mate1             | <code>object</code>                                                                                                                                                                 |                                 | object containing the parameters for mate 1                                     |
| param.mate1.path        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                 | an array with a full mate path (or with a single instance in the expanded tree) |
| param.mate1.csys        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the work coordinate system to use for this first mate                     |
| [param.mate1.flip]      | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>      | flip type which defines the global main axis of the mate (default="Z")          |
| [param.mate1.reorient]  | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>      | orientation around the main axis in 90° steps (default="0")                     |
| param.mate2             | <code>object</code>                                                                                                                                                                 |                                 | object containing the parameters for mate 2                                     |
| param.mate2.path        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                 | an array with a full mate path (or with a single instance in the expanded tree) |
| param.mate2.csys        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                 | id of the work coordinate system to use for this second mate                    |
| [param.mate2.flip]      | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>      | flip type which defines the global main axis of the mate 2 (default="Z")        |
| [param.mate2.reorient]  | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>      | orientation around the main axis in 90° steps (default="0")                     |
| [param.xOffset]         | <code>real</code>                                                                                                                                                                   | <code>0</code>                  | offset along x-axis from mate1 to mate2 (default=0)                             |
| [param.yOffset]         | <code>real</code>                                                                                                                                                                   | <code>0</code>                  | offset along y-axis from mate1 to mate2 (default=0)                             |
| [param.zOffsetLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                 | object defining min and max value of the offset around z-axis                   |
| param.zOffsetLimits.min | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | min value of the offset along z-axis                                            |
| param.zOffsetLimits.max | <code>real</code> \| <code>VOID</code>                                                                                                                                              |                                 | max value value of the offset along z-axis                                      |

**Example**

```js
api.v1.assembly.slider({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs } })
```

<a name="updateSlider"></a>

## updateSlider(param)

> **AGENT NOTE (trained 2026-03-19):** Works with partial params — can update offsets and limits independently without re-specifying mates. Takes constraint id, returns same id.

Updates an existing slider constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated slider constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                   | Type                                                                                                                                                                                | Description                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| param                   | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                     |
| param.id                | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the slider constraint to update                                           |
| [param.name]            | <code>string</code>                                                                                                                                                                 | name for the slider constraint                                                  |
| [param.mate1]           | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                     |
| [param.mate1.path]      | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| [param.mate1.csys]      | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                     |
| [param.mate1.flip]      | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                        |
| [param.mate1.reorient]  | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                   |
| [param.mate2]           | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                     |
| [param.mate2.path]      | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree) |
| [param.mate2.csys]      | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                    |
| [param.mate2.flip]      | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate 2                      |
| [param.mate2.reorient]  | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                   |
| [param.xOffset]         | <code>real</code>                                                                                                                                                                   | offset along x-axis from mate1 to mate2                                         |
| [param.yOffset]         | <code>real</code>                                                                                                                                                                   | offset along y-axis from mate1 to mate2                                         |
| [param.zOffsetLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining min and max value of the offset around z-axis                   |
| param.zOffsetLimits.min | <code>real</code> \| <code>VOID</code>                                                                                                                                              | min value of the offset along z-axis                                            |
| param.zOffsetLimits.max | <code>real</code> \| <code>VOID</code>                                                                                                                                              | max value value of the offset along z-axis                                      |

**Example**

```js
api.v1.assembly.updateSlider({ id: constraint, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs } })
```

<a name="getCircularPattern"></a>

## getCircularPattern(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, instanceId, name, mate1:{csys, flip, path, reorient}, instanceCount, angle, offset}`. Angle in radians. Offset defaults to 0. Query by assembly/instance id + constraint name.

Returns the circular pattern constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    instanceId: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    instanceCount: real
    angle: real
    offset: real
   } | Array<{
    id: id
    instanceId: id
    name: string
    mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    instanceCount: real
    angle: real
    offset: real
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getCircularPattern({ id: 111, name: 'CP1' })
```

<a name="getSlider"></a>

## getSlider(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, name, mate1, mate2, xOffset, yOffset, zOffsetLimits}`. Unconstrained zOffsetLimits shows as `{min: null, max: null}`. Takes `{id: assemblyId, name}`.

Returns the slider constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     xOffset: real
     yOffset: real
    zOffsetLimits: {
       min: real, max: real
     }
   } | Array<{
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     xOffset: real
     yOffset: real
    zOffsetLimits: {
       min: real, max: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getSlider({ id: assembly, name: 'Slider_2' })
```

<a name="getParallel"></a>

## getParallel(param)

> **AGENT NOTE (trained 2026-03-19):** Same pattern as other get* constraint APIs — takes `{ id, name }`, returns full state with all 4 limit sets in radians/mm. Returns `null` if not found.

Returns the parallel constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    xOffsetLimits: {
       min: real, max: real
     }
    yOffsetLimits: {
       min: real, max: real
     }
    zOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | Array<{
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    xOffsetLimits: {
       min: real, max: real
     }
    yOffsetLimits: {
       min: real, max: real
     }
    zOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getParallel({ id: assembly, name: 'Parallel_12' })
```

<a name="getPlanar"></a>

## getPlanar(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, name, mate1, mate2, zOffset, xOffsetLimits, yOffsetLimits, zRotationLimits}`. Unconstrained limits show as `{min: null, max: null}`. Rotation limits returned in radians. Returns `null` with error message if name not found.

Returns the planar constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     zOffset: real
    xOffsetLimits: {
       min: real, max: real
     }
    yOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | Array<{
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     zOffset: real
    xOffsetLimits: {
       min: real, max: real
     }
    yOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getPlanar({ id: assembly, name: 'Planar' })
```

<a name="getCylindrical"></a>

## getCylindrical(param)

> **AGENT NOTE (trained 2026-03-19):** Same pattern as getRevolute — takes `{ id, name }`, returns full state with mate paths, limits in radians. Returns `null` if not found.

Returns the cylindrical constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    zOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | Array<{
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    zOffsetLimits: {
       min: real, max: real
     }
    zRotationLimits: {
       min: real, max: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getCylindrical({ id: instance, name: 'Cylindrical' })
```

<a name="update3DConstraintValue"></a>

## update3DConstraintValue(param)

> **AGENT NOTE (trained 2026-03-19):** Returns null result (VOID). Works on any constraint with offset/rotation values (tested on planar). Supports `X_OFFSET`, `Y_OFFSET`, `Z_OFFSET`, `Z_ROTATION`. Z_ROTATION accepts "45deg" string (stored as radians with expression). Batch update via array param — pass multiple `{id, name, value}` objects. Result returns full structure tree (verbose).

Updates multiple limited values of constraints

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param       | Type                                                                                                                                               | Description                                                                                                                                                             |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param       | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                            | object or objects containing the parameters                                                                                                                             |
| param.id    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                        | id of the constraint to update a value                                                                                                                                  |
| param.name  | <code>&quot;X_OFFSET&quot;</code> \| <code>&quot;Y_OFFSET&quot;</code> \| <code>&quot;Z_OFFSET&quot;</code> \| <code>&quot;Z_ROTATION&quot;</code> | name of the param to update the value                                                                                                                                   |
| param.value | <code>real</code> \| <code>expression</code>                                                                                                       | value to set for the given constraint value, if param.name = "Z_ROTATION" it's possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.update3DConstraintValue({ id: constraint, name: 'X_OFFSET', value: 50 })
api.v1.assembly.update3DConstraintValue({ id: constraint, name: 'Z_ROTATION', value: '135deg' })
```

<a name="convertToTemplate"></a>

## convertToTemplate([param])

Converts the current root assembly into an assembly template
and creates a new root assembly, where the new template can be used
to create instances from.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                | Default                              | Description                                                |
| ------------ | ------------------- | ------------------------------------ | ---------------------------------------------------------- |
| [param]      | <code>object</code> |                                      | object containing the parameters                           |
| [param.name] | <code>string</code> | <code>&quot;Subassembly&quot;</code> | name of the new converted template (default="Subassembly") |

**Example**

```js
api.v1.assembly.convertToTemplate()
```

> **AGENT NOTE (trained 2026-03-19):** Always returns null (VOID). Converts the root assembly into an assembly template and creates a new empty root. After conversion, the original root's id appears in getAssemblyTemplate() results. Part templates survive the conversion. The `name` param sets the template name (default "Subassembly").

<a name="exportNode"></a>

## exportNode(param)

> **AGENT NOTE (trained 2026-03-19):** Works on instances, assembly roots, and templates. Returns `{success: 1, content: string}` as data string when no file/url specified. OFB content is binary (use base64 encoding for safe transport). Supports `compression: 'deflate'` + `encoding: 'base64'` combo — ~6x size reduction for OFB. STP export also works. Template export gives just the part, not instances.

Exports a node from the assembly tree or a template from containers.
A node can be any instance (part or assembly) in the tree.
A template can be a part or assembly, doesn't matter if it has already been instantiated or not.
In any case, if the node or template is a part, the root of the created file is a part.
In case of the node or template is an assembly, the root of the created file is a root assembly.
By default the model is written to a data string.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    success: boolean,
    content?: string  // content is the data string of the export and is only available if neither file nor url is defined.
  }
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                         | Default                      | Description                                                                                                                                                              |
| ------------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| param               | <code>object</code>                                          |                              | object containing the parameters                                                                                                                                         |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code>  |                              | id of the node or template to export                                                                                                                                     |
| [param.format]      | <code>&quot;OFB&quot;</code> \| <code>&quot;STP&quot;</code> | <code>&quot;OFB&quot;</code> | the format of exported data of the node or template (default="OFB"). If stored to a file, format is optional (if missing, the file ending is used to define the format). |
| [param.file]        | <code>string</code>                                          |                              | full path of the file. Path has to be a for the ClassCAD process reachable local or UNC path.                                                                            |
| [param.url]         | <code>string</code>                                          |                              | url to send the model data to.                                                                                                                                           |
| [param.encoding]    | <code>&quot;base64&quot;</code>                              |                              | the encoding the data will be encoded with. If compression is also set, the decoding happens after compression!                                                          |
| [param.compression] | <code>&quot;deflate&quot;</code>                             |                              | the compression algorithm the data is compressed with.                                                                                                                   |

**Example**

```js
api.v1.assembly.exportNode({ id: instance, format: 'STP' })
```

<a name="finishMovingUnderConstraints"></a>

## finishMovingUnderConstraints(param)

> **AGENT NOTE (trained 2026-03-19):** Final step of 3-step MUC workflow. Only needs the assembly id. Returns VOID. Must be called after startMovingUnderConstraints + moveUnderConstraints sequence.

Finishes moving the constrained objects.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param    | Type                                                        | Description                                         |
| -------- | ----------------------------------------------------------- | --------------------------------------------------- |
| param    | <code>object</code>                                         | object containing the parameters                    |
| param.id | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the assembly which contains the moved objects |

**Example**

```js
api.v1.assembly.finishMovingUnderConstraints({ id: assembly })
```

<a name="moveUnderConstraints"></a>

## moveUnderConstraints(param)

> **AGENT NOTE (trained 2026-03-19):** Middle step of the 3-step MUC workflow (start→move→finish). Pass `offset` for translation or `rotation` (basis matrix) for rotation. Can be called multiple times between start/finish. Returns VOID (null). Offsets are absolute from the start position, not cumulative.

Attempts to move constrained objects.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Default                                                        | Description                                                  |
| ------------------- | ----------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------ |
| param               | <code>object</code>                                         |                                                                | object containing the parameters                             |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> |                                                                | id of the assembly which contains the moved objects          |
| [param.rotation]    | <code>object</code>                                         | <code>\{ xDir: [1,0,0], yDir: [0,1,0], zDir: [0,0,1] \}</code> | object containing three consecutive rotation (basis) vectors |
| param.rotation.xDir | <code>point</code>                                          |                                                                | basis vector containing the rotation around x-axis           |
| param.rotation.yDir | <code>point</code>                                          |                                                                | basis vector containing the rotation around y-axis           |
| param.rotation.zDir | <code>point</code>                                          |                                                                | basis vector containing the rotation around z-axis           |
| [param.offset]      | <code>point</code>                                          | <code>[0,0,0]</code>                                           | vector containing the translation                            |

**Example**

```js
api.v1.assembly.moveUnderConstraints({ id: assembly, rotation: { xDir: [0, -1, 0], yDir: [1, 0, 0], zDir: [0, 0, 1] } })
api.v1.assembly.moveUnderConstraints({ id: assembly, offset: [20, 150, 30] })
```

<a name="startMovingUnderConstraints"></a>

## startMovingUnderConstraints(param)

> **AGENT NOTE (trained 2026-03-19):** First step of 3-step MUC workflow. `mucType` options: `TRANSLATION_1D`, `TRANSLATION_2D`, `ROTATION`. `pivotInfo` is the fixed point for rotation. `instanceIds` specifies which instances to move. Returns VOID. Must be followed by moveUnderConstraints calls and finishMovingUnderConstraints.

Prepares to move constrained objects.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                                                                                    | Description                                              |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| param             | <code>object</code>                                                                                                     | object containing the parameters                         |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                             | id of the assembly which contains the moved objects      |
| param.instanceIds | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                            | ids of the instances to be moved                         |
| param.pivotInfo   | <code>point</code>                                                                                                      | this is the fixed point, where the object rotates around |
| param.mucType     | <code>&quot;TRANSLATION_1D&quot;</code> \| <code>&quot;TRANSLATION_2D&quot;</code> \| <code>&quot;ROTATION&quot;</code> | type of transformation                                   |

**Example**

```js
api.v1.assembly.startMovingUnderConstraints({ id: assemblyRoot, instanceIds: [instance], pivotInfo: [50, 50, 20], mucType: 'ROTATION' })
```

<a name="transformInstance"></a>

## transformInstance(param)

Transforms instances by the given transformation.
Transforming an instance means, that the same other instances and the template of them will also be transformed.
Important:

- Left-handed transformation matrices are not yet supported
- Scaling part of the 4x4 matrix will be ignored.
- Matrices must be orthogonal

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                | Type                                                        | Default            | Description                                                                                         |
| -------------------- | ----------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------- |
| param                | <code>object</code> \| <code>Array&lt;object&gt;</code>     |                    | object or objects containing the parameters                                                         |
| param.id             | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id or identifier of the instance to transform                                                       |
| param.transformation | <code>Array&lt;Array&lt;real&gt;&gt;</code>                 |                    | transformation with 4x4 matrix which transforms the given instance relatively from current position |
| [param.isLocal]      | <code>boolean</code>                                        | <code>FALSE</code> | boolean flag to define whether the transformation is local to owner or global(default=FALSE)        |

**Example**

```js
api.v1.assembly.transformInstance({
  id: instance,
  transformation: [
    [1, 0, 0, 50],
    [0, 1, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1],
  ],
})
```

> **AGENT NOTE (trained 2026-03-19):** Relative 4x4 matrix transform — transforms accumulate (two successive calls compound). `isLocal: true` transforms in owner's local coordinate space. Returns null. WARNING: transforming an instance affects ALL instances sharing the same template.

<a name="transformInstanceTo"></a>

## transformInstanceTo(param)

Transforms instances by the given transformation.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                | Type                                                        | Default            | Description                                                                                               |
| -------------------- | ----------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------- |
| param                | <code>object</code> \| <code>Array&lt;object&gt;</code>     |                    | object or objects containing the parameters                                                               |
| param.id             | <code>string</code> \| <code>real</code> \| <code>id</code> |                    | id or identifier of the instance to transform                                                             |
| param.transformation | <code>Array&lt;point&gt;</code>                             |                    | transformation [origin, x-Dir, y-Dir] of the instance in global coordinates to certain absolute position. |
| [param.isLocal]      | <code>boolean</code>                                        | <code>FALSE</code> | boolean flag to define whether the transformation is local to owner or global(default=FALSE)              |

**Example**

```js
api.v1.assembly.transformInstanceTo({
  id: instance,
  transformation: [
    [-10, -10, 0],
    [1, 0, 0],
    [0, 1, 0],
  ],
})
```

> **AGENT NOTE (trained 2026-03-19):** Absolute placement via `[[origin],[xDir],[yDir]]` — replaces current transform entirely (not additive, unlike transformInstance). Returns null.

<a name="loadProduct"></a>

## loadProduct(param)

Loads a product from file, data or url.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { id: string|real|id } | VOID  // object containing id of the loaded product
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                         | Default                      | Description                                                                                                                       |
| ------------------- | ------------------------------------------------------------ | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                          |                              | object containing the parameters                                                                                                  |
| [param.url]         | <code>string</code>                                          |                              | url to load product from                                                                                                          |
| [param.file]        | <code>string</code>                                          |                              | full path of the file. Path has to be a for the ClassCAD process reachable local or UNC path.                                     |
| [param.data]        | <code>string</code>                                          |                              | data/content of the model to load                                                                                                 |
| [param.encoding]    | <code>&quot;base64&quot;</code>                              |                              | the encoding the data is encoded with. If compression is also set, the decoding happens first!                                    |
| [param.compression] | <code>&quot;deflate&quot;</code>                             |                              | the compression algorithm the data is compressed with.                                                                            |
| [param.format]      | <code>&quot;OFB&quot;</code> \| <code>&quot;STP&quot;</code> | <code>&quot;OFB&quot;</code> | content format of to load product (default="OFB"). For load from file, the format is not needed if the extension fits the format. |
| [param.ident]       | <code>string</code>                                          |                              | custom string identifier for the loaded product.                                                                                  |

**Example**

```js
api.v1.assembly.loadProduct({ file: '/var/models/file.stp' })
api.v1.assembly.loadProduct({ url: 'https://.../file.ofb', format: 'OFB' })
api.v1.assembly.loadProduct({ data: 'abc123', format: 'OFB' })
```

> **AGENT NOTE (trained 2026-03-19):** `data` param MUST be a string. If using `common.save()` result, pass `result.content` (not the whole object). `common.save({format:'OFB'})` returns `{content: "...", success: 1}`. Supports OFB and STP formats. Returns `{id: ...}` on success.

<a name="setCurrentInstance"></a>

## setCurrentInstance(param)

Sets the given instance or root assembly as the current. The current product
will also be set to the current instance's product template.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param    | Type                                                        | Description                                                   |
| -------- | ----------------------------------------------------------- | ------------------------------------------------------------- |
| param    | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters                   |
| param.id | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the instance or root assembly to set as the current one |

**Example**

```js
api.v1.assembly.setCurrentInstance({ id: instance })
```

> **AGENT NOTE (trained 2026-03-19):** Returns null (not previous instance). Sets which instance is "active" — the current product will auto-switch to that instance's template. Can pass root assembly id to return to top level.

<a name="setIdent"></a>

## setIdent(param)

> **AGENT NOTE (trained 2026-03-19):** Returns VOID. Works on instances, assemblies, and part templates. Ident must be unique — but duplicates don't throw errors (silently succeeds, may overwrite). Can change an existing ident by calling again with new value. Useful for stable string-based lookups instead of numeric ids.

Sets a string identifier for an existing object

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param       | Type                                                        | Description                                      |
| ----------- | ----------------------------------------------------------- | ------------------------------------------------ |
| param       | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters      |
| param.id    | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the object to set an identifier on         |
| param.ident | <code>string</code>                                         | identifier for the given object. Must be unique. |

**Example**

```js
api.v1.assembly.setIdent({ id: instance, ident: 'ident_526' })
```

<a name="gear"></a>

## gear(param)

> **AGENT NOTE (trained 2026-03-19):** Returns numeric id. Links two revolute/cylindrical constraints via Z_ROTATION coupling. `ratio=2` means constr2 rotates 2× as fast as constr1. `offset` stored in radians; accepts "60deg" string. Negative ratio reverses rotation direction. Default ratio=1, offset=0.

Creates a new gear relation.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created gear relations
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param           | Type                                                        | Default                               | Description                                                                                                                                                      |
| --------------- | ----------------------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param           | <code>object</code> \| <code>Array&lt;object&gt;</code>     |                                       | object or objects containing the parameters                                                                                                                      |
| param.id        | <code>string</code> \| <code>real</code> \| <code>id</code> |                                       | id of the assembly to create the gear relation in                                                                                                                |
| [param.name]    | <code>string</code>                                         | <code>&quot;GearRelation&quot;</code> | name for the gear relation (default="GearRelation")                                                                                                              |
| param.constr1Id | <code>string</code> \| <code>real</code> \| <code>id</code> |                                       | id of the first constraint of the relation                                                                                                                       |
| param.constr2Id | <code>string</code> \| <code>real</code> \| <code>id</code> |                                       | id of the second constraint of the relation                                                                                                                      |
| [param.ratio]   | <code>real</code>                                           | <code>1</code>                        | ratio value of rotational velocity (d(constr2.zRotationValue) / d(constr1.zRotationValue)) (default=1)                                                           |
| [param.offset]  | <code>real</code> \| <code>expression</code>                | <code>0</code>                        | offset value of the second mate angle in radians (default=0). It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.gear({ id: assembly, constr1Id: constraint1, constr2Id: constraint2, ratio: 0.5 })
api.v1.assembly.gear({ id: assembly, constr1Id: constraint1, constr2Id: constraint2, offset: '60deg' })
```

<a name="updateGear"></a>

## updateGear(param)

> **AGENT NOTE (trained 2026-03-19):** Returns the gear id (same as input). Partial update — can change just ratio or just offset without touching the other. Accepts "90deg" string for offset. All params optional except id.

Updates an existing gear relation.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated gear relations
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Description                                                                                                                                          |
| ----------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| param             | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters                                                                                                          |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the gear relation to update                                                                                                                    |
| [param.name]      | <code>string</code>                                         | name for the gear relation                                                                                                                           |
| [param.constr1Id] | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the first constraint of the relation                                                                                                           |
| [param.constr2Id] | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the second constraint of the relation                                                                                                          |
| [param.ratio]     | <code>real</code>                                           | ratio value of rotational velocity (d(constr2.zRotationValue) / d(constr1.zRotationValue))                                                           |
| [param.offset]    | <code>real</code> \| <code>expression</code>                | offset value of the second mate angle in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updateGear({ id: gear, offset: 1.57 })
api.v1.assembly.updateGear({ id: gear, offset: '90deg' })
```

<a name="group"></a>

## group(param)

> **AGENT NOTE (trained 2026-03-19):** Returns numeric id. Logical grouping of instances — does NOT add physical constraints between them. `instanceIds` is the full member list. Default name "Group".

Creates a new group constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created group constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param             | Type                                                        | Default                        | Description                                       |
| ----------------- | ----------------------------------------------------------- | ------------------------------ | ------------------------------------------------- |
| param             | <code>object</code> \| <code>Array&lt;object&gt;</code>     |                                | object or objects containing the parameters       |
| param.id          | <code>string</code> \| <code>real</code> \| <code>id</code> |                                | id of the assembly to create the gear relation in |
| [param.name]      | <code>string</code>                                         | <code>&quot;Group&quot;</code> | name for the group constraint (default="Group")   |
| param.instanceIds | <code>Array&lt;(string\|real\|id)&gt;</code>                |                                | ids of the instances to group                     |

**Example**

```js
api.v1.assembly.group({ id: assembly, instanceIds: [114, 125, 129] })
```

<a name="updateGroup"></a>

## updateGroup(param)

> **AGENT NOTE (trained 2026-03-19):** Returns the group id. `instanceIds` replaces the entire member list (not additive). Can rename via `name` param. Partial update supported — just name, just instanceIds, or both.

Updates an existing group constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated group constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param               | Type                                                        | Description                                 |
| ------------------- | ----------------------------------------------------------- | ------------------------------------------- |
| param               | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the group constraint to update        |
| [param.name]        | <code>string</code>                                         | name for the group constraint               |
| [param.instanceIds] | <code>Array&lt;(string\|real\|id)&gt;</code>                | ids of the instances to group               |

**Example**

```js
api.v1.assembly.updateGroup({ id: group, instanceIds: [114, 129] })
```

<a name="assemblyTemplate"></a>

## assemblyTemplate([param])

Creates a new assembly and adds it as template to the product container.
This assembly can be used for assembly building.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the new assembly template
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param        | Type                | Default                           | Description                                            |
| ------------ | ------------------- | --------------------------------- | ------------------------------------------------------ |
| [param]      | <code>object</code> |                                   | object containing all the parameters                   |
| [param.name] | <code>string</code> | <code>&quot;Assembly&quot;</code> | name of the new assembly template (default="Assembly") |

**Example**

```js
api.v1.assembly.assemblyTemplate()
```

> **AGENT NOTE (trained 2026-03-19):** Returns numeric id. Default name "Assembly". Used for sub-assembly definitions (hierarchical assemblies). Requires assembly.create() first. Can create multiple assembly templates per session.

<a name="from"></a>

## from(param)

Creates an assembly from a json defined assembly or an ecxml definition.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created root assembly
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param          | Type                                                                                            | Description                                                                                   |
| -------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| param          | <code>object</code>                                                                             | object containing all the parameters                                                          |
| [param.url]    | <code>string</code>                                                                             | url to load from                                                                              |
| [param.file]   | <code>string</code>                                                                             | full path of the file. Path has to be a for the ClassCAD process reachable local or UNC path. |
| [param.data]   | <code>string</code>                                                                             | data/content to load from                                                                     |
| [param.format] | <code>&quot;JSON&quot;</code> \| <code>&quot;XML&quot;</code> \| <code>&quot;ECXML&quot;</code> | content format of to load from, has to be passed if load from stream or url                   |

**Example**

```js
api.v1.assembly.from({ file: '/var/models/file.ecxml' })
api.v1.assembly.from({ url: 'https://.../file.ecxml', format: 'ECXML' })
api.v1.assembly.from({ data: 'abc123', format: 'JSON' })
```

> **AGENT NOTE (trained 2026-03-19):** Returns root assembly id even on partial errors. JSON format requires ClassCAD's specific assembly schema (not arbitrary JSON) — ad-hoc structures cause template/instance/constraint creation errors. ECXML is the most reliable format. Creates a complete assembly (including root), so do NOT call `assembly.create()` first.

<a name="getSpherical"></a>

## getSpherical(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, name, mate1, mate2, yRotationLimits: {max}}`. Only `max` (no min) — cone-of-freedom model. Unconstrained shows `{max: null}`. Rotation in radians. Takes `{id: assemblyId, name}`.

Returns the spherical constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    yRotationLimits: {
       max: real
     }
   } | Array<{
    id: id
    name: string
     mate1: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
     mate2: {
       path: id[],
       csys: id,
       flip: "X" | "-X" | "Y" | "-Y" | "Z" | "-Z",
       reorient: "0" | "90" | "180" | "270"
     }
    yRotationLimits: {
       max: real
     }
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getSpherical({ id: assembly, name: 'Spherical2' })
```

<a name="getGroup"></a>

## getGroup(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, instanceIds, name}`. Lookup by assembly id + group name.

Returns the group constraint of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
     instanceIds: id[]
    name: string
   } | Array<{
    id: id
     instanceIds: id[]
    name: string
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                          |
| ---------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters          |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for constraint |
| param.name | <code>string</code>                                         | the name of the constraint to look for               |

**Example**

```js
api.v1.assembly.getGroup({ id: assembly, name: 'Group_Left' })
```

<a name="createUncommitedObject"></a>

## createUncommitedObject(param)

> **AGENT NOTE (trained 2026-03-19):** Creates empty placeholder constraint objects. Valid types: CC_FastenedConstraint, CC_RevoluteConstraint, CC_CylindricalConstraint, CC_PlanarConstraint, CC_SliderConstraint, CC_SphericalConstraint, CC_ParallelConstraint. Returns the object id. Invalid type → null + ERROR with C++ trace. Use case: pre-creating constraint shells for later population via update APIs.

Attention: This method should only be used, if the intention is very clear.
Creates a new uncommited (empty) object in the given assembly.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the created object
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                           |
| ---------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| param      | <code>object</code>                                         | object containing all the parameters                  |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the assembly to create the uncommited object in |
| param.type | <code>string</code>                                         | the type of object to be created                      |
| param.name | <code>string</code>                                         | the name of the object to be created                  |

**Example**

```js
api.v1.assembly.createUncommitedObject({ id: assembly, type: 'CC_FastenedConstraint', name: 'Fastened' })
```

<a name="getGear"></a>

## getGear(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{id, name, constr1Id, constr2Id, ratio, offset}`. Offset always returned as radians (even if set via "60deg"). Lookup by assembly id + gear name.

Returns the gear relation of given reference with specified name.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: {
    id: id
    name: string
     constr1Id: id
     constr2Id: id
    ratio: real
    offset: real
   } | Array<{
    id: id
    name: string
     constr1Id: id
     constr2Id: id
    ratio: real
    offset: real
   } | VOID> | VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                            |
| ---------- | ----------------------------------------------------------- | ------------------------------------------------------ |
| param      | <code>object</code> \| <code>Array&lt;object&gt;</code>     | object or objects containing the parameters            |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product or instance to look for the relation |
| param.name | <code>string</code>                                         | the name of the relation to look for                   |

**Example**

```js
api.v1.assembly.getGear({ id: assembly, name: 'Gear_Left' })
```

<a name="spherical"></a>

## spherical(param)

> **AGENT NOTE (trained 2026-03-19):** 3-DOF ball joint constraint. `yRotationLimits.max` defines cone half-angle from Z axis (no min — symmetric cone). Accepts "45deg" string or raw radians (1.0472). Stored as radians. Default max=null (unconstrained = full sphere of rotation).

Creates a new spherical constraint.
If optional parameters are not set, the default values will be used, see (default=xy).

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the created spherical constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Default                            | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             |                                    | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                    | id of the assembly to create the spherical constraint in                                                                                           |
| [param.name]              | <code>string</code>                                                                                                                                                                 | <code>&quot;Spherical&quot;</code> | name for the spherical constraint (default="Spherical")                                                                                            |
| param.mate1               | <code>object</code>                                                                                                                                                                 |                                    | object containing the parameters for mate 1                                                                                                        |
| param.mate1.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                    | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate1.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                    | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>         | flip type which defines the global main axis of the mate (default="Z")                                                                             |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>         | orientation around the main axis in 90° steps (default="0")                                                                                        |
| param.mate2               | <code>object</code>                                                                                                                                                                 |                                    | object containing the parameters for mate 2                                                                                                        |
| param.mate2.path          | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        |                                    | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| param.mate2.csys          | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         |                                    | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | <code>&quot;Z&quot;</code>         | flip type which defines the global main axis of the mate 2 (default="Z")                                                                           |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | <code>&quot;0&quot;</code>         | orientation around the main axis in 90° steps (default="0")                                                                                        |
| [param.yRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            |                                    | object defining a max value of rotation around y-axis                                                                                              |
| param.yRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   |                                    | max value of rotation around y-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.spherical({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs } })
api.v1.assembly.spherical({ id: assembly, mate1: { path: [instance], csys: wcs }, mate2: { path: [instance2], csys: wcs }, yRotationLimits: { max: '45deg' } })
```

<a name="updateSpherical"></a>

## updateSpherical(param)

> **AGENT NOTE (trained 2026-03-19):** Works with partial params — can update just yRotationLimits without mates. Takes constraint id, returns same id. Accepts "90deg" strings.

Updates an existing spherical constraint.
If optional parameters are not set, the constraint will keep the existing values.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // id or ids of the updated spherical constraints
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                | Description                                                                                                                                        |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                     | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                             | object or objects containing the parameters                                                                                                        |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the spherical constraint to update                                                                                                           |
| [param.name]              | <code>string</code>                                                                                                                                                                 | name for the spherical constraint                                                                                                                  |
| [param.mate1]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 1                                                                                                        |
| [param.mate1.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate1.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this first mate                                                                                        |
| [param.mate1.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate                                                                                           |
| [param.mate1.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.mate2]             | <code>object</code>                                                                                                                                                                 | object containing the parameters for mate 2                                                                                                        |
| [param.mate2.path]        | <code>Array&lt;(string\|real\|id)&gt;</code>                                                                                                                                        | an array with a full mate path (or with a single instance in the expanded tree)                                                                    |
| [param.mate2.csys]        | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                         | id of the work coordinate system to use for this second mate                                                                                       |
| [param.mate2.flip]        | <code>&quot;X&quot;</code> \| <code>&quot;-X&quot;</code> \| <code>&quot;Y&quot;</code> \| <code>&quot;-Y&quot;</code> \| <code>&quot;Z&quot;</code> \| <code>&quot;-Z&quot;</code> | flip type which defines the global main axis of the mate 2                                                                                         |
| [param.mate2.reorient]    | <code>&quot;0&quot;</code> \| <code>&quot;90&quot;</code> \| <code>&quot;180&quot;</code> \| <code>&quot;270&quot;</code>                                                           | orientation around the main axis in 90° steps                                                                                                      |
| [param.yRotationLimits]   | <code>object</code> \| <code>VOID</code>                                                                                                                                            | object defining a max value of rotation around y-axis                                                                                              |
| param.yRotationLimits.max | <code>real</code> \| <code>expression</code> \| <code>VOID</code>                                                                                                                   | max value of rotation around y-axis in radians. It's also possible to provide an angle expression in degrees as string like "45deg", "135deg", ... |

**Example**

```js
api.v1.assembly.updateSpherical({ id: constraint, mate1: { path: [instance], csys: wcs } })
api.v1.assembly.updateSpherical({ id: constraint, yRotationLimits: { max: 1.57 } })
```

<a name="setCurrentProduct"></a>

## setCurrentProduct(param)

Sets the current product. The current product is exported in the save commando.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id | VOID  // id of the product which was the current one before calling this function
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param    | Type                                                        | Description                                 |
| -------- | ----------------------------------------------------------- | ------------------------------------------- |
| param    | <code>object</code>                                         | object containing the parameters            |
| param.id | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product to become the current one |

**Example**

```js
api.v1.assembly.setCurrentProduct({ id: assembly })
```

> **AGENT NOTE (trained 2026-03-19):** Returns the PREVIOUS current product id — useful for save/restore patterns. The current product is what gets exported by `common.save`.

<a name="calculateMassProperties"></a>

## calculateMassProperties(param)

> **AGENT NOTE (trained 2026-03-19):** Returns `{cog: {x,y,z}, volume: real}`. COG in world coordinates. Works on instances, assembly roots, and templates. Assembly-level = sum of all solids (volume additive, COG weighted average). Cylinder πr²h matches to ~0.01% (tessellation-dependent). No mass/density — volume only.

Calculates the center of gravity (cog) and the volume of the given object.
Depending on the input parameter the calculations were made
on a single solid, a part, a subassembly, the whole model, ...
E.g. the center of gravity of a root assembly, will be the sum of all solid's center of gravity
over the whole assembly structure. If a solid is given, the center of gravity will be local to its part.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: { cog: point, volume: real } | VOID  // object containing the center of gravity and the volume
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param    | Type                                                        | Description                                      |
| -------- | ----------------------------------------------------------- | ------------------------------------------------ |
| param    | <code>object</code>                                         | object containing the parameters                 |
| param.id | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the object to get the mass properties from |

**Example**

```js
api.v1.assembly.calculateMassProperties({ id: assemblyRoot })
```

<a name="deleteTemplate"></a>

## deleteTemplate(param)

Deletes a part or assembly template.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param     | Type                                         | Description                          |
| --------- | -------------------------------------------- | ------------------------------------ |
| param     | <code>object</code>                          | object containing all the parameters |
| param.ids | <code>Array&lt;(string\|real\|id)&gt;</code> | ids of the templates to delete       |

**Example**

```js
api.v1.assembly.deleteTemplate({ ids: [796, 852, 963] })
```

> **AGENT NOTE (trained 2026-03-19):** Always returns null. Works on both part and assembly templates. **WARNING: Deletes templates even if they have active instances — no error, no protection, cascade-deletes silently.** Supports batch deletion via ids array.

<a name="getWorkGeometry"></a>

## getWorkGeometry(param)

> **AGENT NOTE (trained 2026-03-19):** Looks up work geometry (WCS, work plane, work axis, work point) by name string. Works on **instances** (returns the id from the underlying template). Does NOT work on assembly root — returns null + ERROR code 1015. Returns null + error for non-existent names.

Returns the id of the work geometry object with the given name from the given assembly or instance.
Work geometry can be a workpoint, -axis, -plane or -coordinate system.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID  // id of the found work geometry
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param      | Type                                                        | Description                                             |
| ---------- | ----------------------------------------------------------- | ------------------------------------------------------- |
| param      | <code>object</code>                                         | object containing all the parameters                    |
| param.id   | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the assembly or instance to get the geometry from |
| param.name | <code>string</code>                                         | the name of the work geometry to look for               |

**Example**

```js
api.v1.assembly.getWorkGeometry({ id: instance, name: 'WorkCSys_Top' })
```

<a name="deleteConstraint"></a>

## deleteConstraint(param)

> **AGENT NOTE (trained 2026-03-19):** Returns VOID. Works on constraints, gear relations, and group constraints — all are deletable via `ids` array. Batch delete supported. Instances remain after their constraints are deleted (they become unconstrained).

Deletes constraints/ relations from assemblies.

**Kind**: v1.assembly function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param     | Type                                         | Description                                               |
| --------- | -------------------------------------------- | --------------------------------------------------------- |
| param     | <code>object</code>                          | object containing all the parameters                      |
| param.ids | <code>Array&lt;(string\|real\|id)&gt;</code> | ids or identifiers of the constraints/relations to delete |

**Example**

```js
api.v1.assembly.deleteConstraint({ ids: [constraint1, constraint2] })
```
