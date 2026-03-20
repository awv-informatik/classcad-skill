# Drawing2D API Reference — `api.v1.drawing2d.*`

> 2D technical drawing views, dimensions, and DXF/SVG export from 3D models.

## Table of Contents

### Views
- [view](#view) — Create 2D views of a product (TOP, FRONT, RIGHT, etc.)
- [centerView](#centerview) — Center views to origin based on boundary boxes
- [placeView](#placeview) — Place views with relative offset
- [getBoundaryBoxFromView](#getboundaryboxfromview) — Get min/max boundary box of views

### Dimensions
- [dimension](#dimension) — Create dimensions (LINEAR, ANGULAR, RADIAL, DIAMETER)
- [deleteDimension](#deletedimension) — Delete dimensions
- [updateDimensionPosition](#updatedimensionposition) — Update dimension text position

### Export
- [exportDXF](#exportdxf) — Export views to DXF format
- [exportSVG](#exportsvg) — Export views to SVG format
- [isDXFAvailable](#isdxfavailable) — Check if DXF export is available
- [isSVGAvailable](#issvgavailable) — Check if SVG export is available

---

> **AGENT HINTS**:
> - **Workflow**: Create dimensions → create views → center views → place views → export DXF/SVG.
> - Dimensions are tied to a `viewType` (e.g., `"FRONT"`, `"TOP"`) and appear only in matching views.
> - View types: `TOP`, `FRONT`, `RIGHT`, `LEFT`, `BOTTOM`, `RIGHT_90`, `LEFT_90`, `BACK`, `ISO`.
> - **`dimension` value placeholders**: `"<>"` is replaced by auto-calculated value. `"%%d"` → °, `"%%c"` → Ø.

---
<a name="dimension"></a>

## dimension(param)

Creates one or multiple dimensions in the product.
The dimensions will be visible after creating and exporting the views to svg or dxf.

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|VOID|Array<id|VOID>  // one or multiple ids of the created dimensions
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                    | Type                                                                                                                                                                                                                                                                                                           | Default                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param                    | <code>object</code> \| <code>Array&lt;object&gt;</code>                                                                                                                                                                                                                                                        |                                    | object or objects containing the parameters                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| param.id                 | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                                                                                                                                                    |                                    | id of the product to create the dimension in                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| param.viewType           | <code>&quot;TOP&quot;</code> \| <code>&quot;FRONT&quot;</code> \| <code>&quot;RIGHT&quot;</code> \| <code>&quot;LEFT&quot;</code> \| <code>&quot;BOTTOM&quot;</code> \| <code>&quot;RIGHT_90&quot;</code> \| <code>&quot;LEFT_90&quot;</code> \| <code>&quot;BACK&quot;</code> \| <code>&quot;ISO&quot;</code> |                                    | type of the view where the dimension will be visible later in the dxf file                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| param.common             | <code>object</code>                                                                                                                                                                                                                                                                                            |                                    | object containing the common parameters                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| param.common.type        | <code>&quot;LINEAR&quot;</code> \| <code>&quot;ANGULAR&quot;</code> \| <code>&quot;RADIAL&quot;</code> \| <code>&quot;DIAMETER&quot;</code>                                                                                                                                                                    |                                    | type of the dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [param.common.name]      | <code>string</code>                                                                                                                                                                                                                                                                                            | <code>&quot;Dimension&quot;</code> | name of the dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [param.common.label]     | <code>string</code>                                                                                                                                                                                                                                                                                            |                                    | label for the dimension text, this will appear in the 2d view next to the dimension e.g. "width = "                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| [param.common.value]     | <code>string</code> \| <code>real</code>                                                                                                                                                                                                                                                                       |                                    | value for the dimension text which will be concatenated with the label, if empty the value will be calculated automatically strings: - If a string is provided, "\<\>" can be used as a placeholder which will be replaced by the current value. - In angular dimensions, "\<\>" will be replaced by an angle value in degrees and °, e.g. "60°" - In radial dimensions, "\<\>" will be replaced by R and the radius value, e.g. "R60" - In diameter dimensions, "\<\>" will be replaced by Ø and the diameter value, e.g. "Ø120" special symbols: - "%%d" will be replaced by °. - "%%c" will be replaced by Ø |
| [param.common.color]     | <code>real</code>                                                                                                                                                                                                                                                                                              | <code>256</code>                   | color of the dimension (autocad color index) in range of [0,256], whereas 256 means the color will be taken from the defined layer in the dxf template                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [param.common.layer]     | <code>string</code>                                                                                                                                                                                                                                                                                            | <code>&quot;1&quot;</code>         | layer where the dimension will appear, e.g. "1", depends on layer definitions in dxf template                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| param.common.textPos     | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position where the dimension text (combination of label and value) will be placed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [param.linear]           | <code>object</code>                                                                                                                                                                                                                                                                                            |                                    | object containing the parameters for a linear dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| param.linear.startPos    | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the start of the linear dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| param.linear.endPos      | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the end of the linear dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [param.linear.textAngle] | <code>real</code>                                                                                                                                                                                                                                                                                              |                                    | If not set, it is calculated depending on orientation, which is OK for most cases. If angle (in radians) is set, it defines the dimension line orientation measured counter clockwise from the x-axis.                                                                                                                                                                                                                                                                                                                                                                                                          |
| param.linear.orientation | <code>&quot;VERTICAL&quot;</code> \| <code>&quot;HORIZONTAL&quot;</code> \| <code>&quot;ALIGNED&quot;</code>                                                                                                                                                                                                   |                                    | orientation of the linear dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| [param.angular]          | <code>object</code>                                                                                                                                                                                                                                                                                            |                                    | object containing the parameters for an angular dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| param.angular.startPos   | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the start of the angular dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| param.angular.endPos     | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the end of the angular dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| param.angular.cornerPos  | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the corner of the angular dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [param.angular.isCCW]    | <code>boolean</code>                                                                                                                                                                                                                                                                                           | <code>TRUE</code>                  | if true the orientation is counter clockwise (ccw) (default=TRUE)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [param.radial]           | <code>object</code>                                                                                                                                                                                                                                                                                            |                                    | object containing the parameters for a radial dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| param.radial.centerPos   | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the center of the radial dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| param.radial.radius      | <code>real</code>                                                                                                                                                                                                                                                                                              |                                    | radius of the radial dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [param.diameter]         | <code>object</code>                                                                                                                                                                                                                                                                                            |                                    | object containing the parameters for a diameter dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| param.diameter.centerPos | <code>point</code>                                                                                                                                                                                                                                                                                             |                                    | position of the center of the diameter dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| param.diameter.radius    | <code>real</code>                                                                                                                                                                                                                                                                                              |                                    | radius of the diameter dimension                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

**Example**

```js
api.v1.drawing2d.dimension({ id: part, common: { type: 'LINEAR', textPos: [25, 20, 0] }, linear: { startPos: [0, 0, 0], endPos: [50, 0, 0], orientation: 'HORIZONTAL' }, viewType: 'FRONT' })
```

> **AGENT NOTE (trained 2026-03-19):** Returns a numeric ID for a single dimension, or an array of IDs for batch (array param) input. All dimension types (LINEAR, RADIAL, DIAMETER, ANGULAR) confirmed working.

> **AGENT NOTE (trained 2026-03-19):** VERTICAL orientation warns (level 41) if start/end points have no separation along the measured axis — "dimension could not be calculated or the dimension value is 0". The dimension is still created.

> **AGENT NOTE (trained 2026-03-19):** DIAMETER type: if `textPos` projects to the same XY as `centerPos`, ClassCAD auto-offsets textPos by 0.001 in X (level-51 warning). Avoid placing textPos directly above/below center.

> **AGENT NOTE (trained 2026-03-19):** `label` + `value: '<>'` works as documented — `<>` is replaced by the auto-calculated value in the exported view.

> **AGENT NOTE (trained 2026-03-19):** Internal storage: dimensions become `CC_LinearDimension`, `CC_AngularDimension`, etc. under the part's `DimensionSet`. The `dxfView` member maps viewType to an integer (TOP=1, FRONT=2).

<a name="centerView"></a>

## centerView(param)

This method centers all given views to origin, depending on each view's boundary box.
This can be useful after creating the views and before placing them

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param         | Type                                                                                                                                                                                                  | Description                                                  |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| param         | <code>object</code>                                                                                                                                                                                   | object containing the parameters                             |
| param.id      | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                                           | id of the product to center the views from                   |
| [param.types] | <code>Array&lt;(&quot;TOP&quot;\|&quot;FRONT&quot;\|&quot;RIGHT&quot;\|&quot;LEFT&quot;\|&quot;BOTTOM&quot;\|&quot;RIGHT_90&quot;\|&quot;LEFT_90&quot;\|&quot;BACK&quot;\|&quot;ISO&quot;)&gt;</code> | type of views to center, if empty all views will be centered |

**Example**

```js
api.v1.drawing2d.centerView({ id: part, types: ['TOP', 'ISO'] })
```

> **AGENT NOTE (trained 2026-03-19):** Returns null (void). Centers views symmetrically around origin based on their bounding boxes. E.g., a TOP view with bbox [(−50, 0), (50, 60)] becomes [(−50, −30), (50, 30)] after centering. Call after `view()` and before `placeView()`.

<a name="view"></a>

## view(param)

Creates the defined 2d views of the given product (solids) in xy-plane and returns the ids of the created views.
Previously created dimensions will appear in the defined views (depends on type),
e.g. a dimension with viewType = "TOP", will appear in the "TOP" view

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: id|Array<id>  // one or multiple ids of the newly created views
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param         | Type                                                                                                                                                                                                  | Default                    | Description                                                                                                                                                   |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| param         | <code>object</code>                                                                                                                                                                                   |                            | object containing the parameters                                                                                                                              |
| param.id      | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                                           |                            | id of the product to create views of                                                                                                                          |
| param.types   | <code>Array&lt;(&quot;TOP&quot;\|&quot;FRONT&quot;\|&quot;RIGHT&quot;\|&quot;LEFT&quot;\|&quot;BOTTOM&quot;\|&quot;RIGHT_90&quot;\|&quot;LEFT_90&quot;\|&quot;BACK&quot;\|&quot;ISO&quot;)&gt;</code> |                            | type of views to create                                                                                                                                       |
| [param.color] | <code>real</code>                                                                                                                                                                                     | <code>0</code>             | color of the view (autocad color index) in range of [0,256], whereas 256 means the color will be taken from the defined layer in the dxf template (default=0) |
| [param.layer] | <code>string</code>                                                                                                                                                                                   | <code>&quot;0&quot;</code> | layer where the view will appear later in the dxf export, it depends on layer definitions in dxf template (default="0\*)                                      |

**Example**

```js
api.v1.drawing2d.view({ id: part, types: ['RIGHT', 'BOTTOM'], color: 125, layer: '5' })
```

> **AGENT NOTE (trained 2026-03-19):** Returns an array of IDs even for a single view type (e.g. `[97]`). View creation requires actual BRep geometry (solids) on the part — without solids, a level-51 error fires: "There must be at least one brep to perform a projection".

> **AGENT NOTE (trained 2026-03-19):** Views are stored as `CC_View2D` objects under `CC_ViewSet`. Each view gets its own `DimensionSet` child containing copies of dimensions matching that viewType. Dimensions must be created BEFORE views for them to appear.

<a name="exportDXF"></a>

## exportDXF(param)

Exports all the views from given product into dxf. By default the model is written to a data string.

**Kind**: v1.drawing2d function  
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

| Param               | Type                                                        | Description                                                                                                     |
| ------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                         | object containing the parameters                                                                                |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the product to export the views from                                                                      |
| [param.file]        | <code>string</code>                                         | full path of the file. Path has to be a for the ClassCAD process reachable local or UNC path.                   |
| [param.url]         | <code>string</code>                                         | url to send the model data to.                                                                                  |
| [param.encoding]    | <code>&quot;base64&quot;</code>                             | the encoding the data will be encoded with. If compression is also set, the decoding happens after compression! |
| [param.compression] | <code>&quot;deflate&quot;</code>                            | the compression algorithm the data is compressed with.                                                          |

**Example**

```js
api.v1.drawing2d.exportDXF({ id: part, file: '/var/models/file.dxf' })
api.v1.drawing2d.exportDXF({ id: part, compression: 'deflate', encoding: 'base64' })
```

> **AGENT NOTE (trained 2026-03-19):** When DXF export is not available (isDXFAvailable=0), this returns `0` (not the documented `{success, content}` object) with message "Export DXF is not supported." Check `isDXFAvailable()` first.

<a name="exportSVG"></a>

## exportSVG(param)

Exports all the views from given product into svg. By default the model is written to a data string.

**Kind**: v1.drawing2d function  
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

| Param               | Type                                                                                         | Description                                                                                                     |
| ------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| param               | <code>object</code>                                                                          | object containing the parameters                                                                                |
| param.id            | <code>string</code> \| <code>real</code> \| <code>id</code>                                  | id of the product to export the views from                                                                      |
| [param.file]        | <code>string</code>                                                                          | full path of the file. Path has to be a for the ClassCAD process reachable local or UNC path.                   |
| [param.url]         | <code>string</code>                                                                          | url to send the model data to.                                                                                  |
| [param.encoding]    | <code>&quot;base64&quot;</code>                                                              | the encoding the data will be encoded with. If compression is also set, the decoding happens after compression! |
| [param.compression] | <code>&quot;deflate&quot;</code>                                                             | the compression algorithm the data is compressed with.                                                          |
| [param.modus]       | <code>&quot;SVG_WHOLE_DRAWING&quot;</code> \| <code>&quot;SVG_FIT_WHOLE_DRAWING&quot;</code> | modus to define how drawing will fit into svg file                                                              |
| [param.renderSize]  | <code>object</code>                                                                          | object containing the render size of the drawing (default=1024x768)                                             |
| param.renderSize.x  | <code>real</code>                                                                            | horizontal size of the drawing                                                                                  |
| param.renderSize.y  | <code>real</code>                                                                            | vertical size of the drawing                                                                                    |

**Example**

```js
api.v1.drawing2d.exportSVG({ id: part, file: '/var/models/file.svg' })
api.v1.drawing2d.exportSVG({ id: part, compression: 'deflate', encoding: 'base64' })
```

> **AGENT NOTE (trained 2026-03-19):** Same behavior as exportDXF when unavailable — returns `0` with "Export SVG is not supported." Check `isSVGAvailable()` first.

<a name="getBoundaryBoxFromView"></a>

## getBoundaryBoxFromView(param)

This method returns min and max point for each given view.

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: Array<{ min: point, max: point }> | VOID  // array of objects containing the min and max point
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param       | Type                                                                                                                                                                                                  | Description                                                                                             |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| param       | <code>object</code>                                                                                                                                                                                   | object containing the parameters                                                                        |
| param.id    | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                                           | id of the product to get boundary boxes of the views from                                               |
| param.types | <code>Array&lt;(&quot;TOP&quot;\|&quot;FRONT&quot;\|&quot;RIGHT&quot;\|&quot;LEFT&quot;\|&quot;BOTTOM&quot;\|&quot;RIGHT_90&quot;\|&quot;LEFT_90&quot;\|&quot;BACK&quot;\|&quot;ISO&quot;)&gt;</code> | type of views to get boundary box from, if empty, boundary boxes of all existing views will be returned |

**Example**

```js
api.v1.drawing2d.getBoundaryBoxFromView({ id: part, types: ['LEFT'] })
```

> **AGENT NOTE (trained 2026-03-19):** Returns array of `{min, max}` objects. Only returns entries for views that actually exist — if you request 4 types but only 3 exist, you get 3 entries (no null/VOID placeholders). Bbox Z values are effectively 0 (2D plane).

<a name="placeView"></a>

## placeView(param)

Places each view relatively to its current position in xy-plane

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param                     | Type                                                                                                                                                                                                                                                                                                           | Description                                                   |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| param                     | <code>object</code>                                                                                                                                                                                                                                                                                            | object containing the parameters                              |
| param.id                  | <code>string</code> \| <code>real</code> \| <code>id</code>                                                                                                                                                                                                                                                    | id of the product to place the views                          |
| param.placements          | <code>Array&lt;object&gt;</code>                                                                                                                                                                                                                                                                               | objects containing the placement information                  |
| param.placements[].type   | <code>&quot;TOP&quot;</code> \| <code>&quot;FRONT&quot;</code> \| <code>&quot;RIGHT&quot;</code> \| <code>&quot;LEFT&quot;</code> \| <code>&quot;BOTTOM&quot;</code> \| <code>&quot;RIGHT_90&quot;</code> \| <code>&quot;LEFT_90&quot;</code> \| <code>&quot;BACK&quot;</code> \| <code>&quot;ISO&quot;</code> | type of view to place by offset vector                        |
| param.placements[].offset | <code>point</code>                                                                                                                                                                                                                                                                                             | offset vector to move view relatively to its current position |

**Example**

```js
api.v1.drawing2d.placeView({ id: part, placements: [{ type: 'TOP', offset: [0, 150, 0] }] })
```

> **AGENT NOTE (trained 2026-03-19):** Returns null (void). Offsets are RELATIVE to current position. If a requested view type doesn't exist, a level-51 warning fires ("View of type: X does not exist in ViewSet!") but doesn't fail the call.

<a name="deleteDimension"></a>

## deleteDimension(param)

Deletes one or multiple dimensions of the 2d view

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param     | Type                                         | Description                                  |
| --------- | -------------------------------------------- | -------------------------------------------- |
| param     | <code>object</code>                          | object containing all the parameters         |
| param.ids | <code>Array&lt;(string\|real\|id)&gt;</code> | ids of the dimensions to remove from 2d view |

**Example**

```js
api.v1.drawing2d.deleteDimension({ ids: [58, 96] })
```

> **AGENT NOTE (trained 2026-03-19):** Returns null (void). Successfully removes dimensions by ID. Views must be recreated after deletion for the change to be reflected in exports.

<a name="updateDimensionPosition"></a>

## updateDimensionPosition(param)

Updates the position of the dimension text.
Attention: The view needs to be recreated to have updated positions in the view

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: VOID
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

| Param     | Type                                                        | Description                                     |
| --------- | ----------------------------------------------------------- | ----------------------------------------------- |
| param     | <code>object</code>                                         | object containing all the parameters            |
| param.id  | <code>string</code> \| <code>real</code> \| <code>id</code> | id of the dimension to change the text position |
| param.pos | <code>point</code>                                          | position of the dimension text to update        |

**Example**

```js
api.v1.drawing2d.updateDimensionPosition({ id: dimension, pos: [50, 60, 0] })
```

> **AGENT NOTE (trained 2026-03-19):** Returns null (void). Updates the textPos of the dimension. As documented, views MUST be recreated (call `view()` again) for the updated position to appear in exports.

<a name="isDXFAvailable"></a>

## isDXFAvailable()

Returns true if DXF functionality is available, false if not.

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: boolean // true if DXF functionality is available
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

**Example**

```js
api.v1.drawing2d.isDXFAvailable()
```

> **AGENT NOTE (trained 2026-03-19):** Returns `0` (falsy) when DXF is not available on the instance. This is a build/license-dependent feature — always check before attempting export.

<a name="isSVGAvailable"></a>

## isSVGAvailable()

Returns true if SVG functionality is available, false if not.

**Kind**: v1.drawing2d function  
**Returns**: <code>object</code> - object containing result and optional messages

```
{
  result: boolean // true if SVG functionality is available
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

**Example**

```js
api.v1.drawing2d.isSVGAvailable()
```

> **AGENT NOTE (trained 2026-03-19):** Returns `0` (falsy) when SVG is not available. Same as isDXFAvailable — build/license-dependent.
