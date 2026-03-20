# ClassCAD API Skill

> **Source**: https://classcad.ch/docs/ · API version: **v1**

## Overview

ClassCAD is a **headless, programmable CAD engine** for building web-based, cloud-based, and automated CAD applications. It is **not** an end-user application — it is designed to be embedded into products, services, and workflows.

Client applications interact with ClassCAD **exclusively through public APIs and API wrappers**. All API calls follow the pattern:

```js
api.v1.<domain>.<method>(param)
```

### Key Characteristics

- **Deterministic execution** — same input always produces the same CAD model
- **Headless** — no UI, no interaction logic, no client assumptions
- **Stable APIs** — external contracts remain stable even as internals evolve
- **Plugin-based** — geometry kernel, constraint solving, etc. provided via plugins

### Deployment Modes

| Mode | Description |
|------|-------------|
| **Proxy-based** | ClassCAD runs as an isolated C++ process behind a proxy (`@classcad/node`) |
| **WebAssembly** | ClassCAD runtime compiled to WASM, runs in browser (typically in a Web Worker) |

---

## API Domains

The v1 API is organized into **7 domains**. Each has its own reference file in this skill:

| Domain | Namespace | Reference | Description |
|--------|-----------|-----------|-------------|
| **Assembly** | `api.v1.assembly.*` | [references/assembly.md](references/assembly.md) | Assembly building: templates, instances, constraints, patterns |
| **Common** | `api.v1.common.*` | [references/common.md](references/common.md) | Session management, load/save, settings, appearance, user data |
| **Curve** | `api.v1.curve.*` | [references/curve.md](references/curve.md) | 2D/3D curve creation in shape containers |
| **Drawing2D** | `api.v1.drawing2d.*` | [references/drawing2d.md](references/drawing2d.md) | 2D views, dimensions, DXF/SVG export |
| **Part** | `api.v1.part.*` | [references/part.md](references/part.md) | Feature-based part modeling (primitives, booleans, patterns, sketches) |
| **Sketch** | `api.v1.sketch.*` | [references/sketch.md](references/sketch.md) | 2D constrained sketches on work planes |
| **Solid** | `api.v1.solid.*` | [references/solid.md](references/solid.md) | Direct solid modeling within entity injection features |

---

## Common Conventions

### Return Shape

Every API call returns an object with this structure:

```ts
{
  result: <varies>   // API-specific result (id, array, VOID, etc.)
  messages?: { message: string, level: real, code: real, api: string }[]
  maxLevel?: real
}
```

> **AGENT HINT**: Always check `messages` for warnings/errors. `maxLevel` indicates the highest severity level.

### Parameter Conventions

- **`[param]`** = optional parameter (square brackets in the signature)
- **`param`** = required parameter
- **`id`** types accept `string | real | id` — they are ClassCAD object identifiers
- **`point`** = `[x, y, z]` — a 3-element array of reals
- **`expression`** = a string like `'@expr.myVar'` or `'sin(C:PI/2)'`
- **Angle expressions**: strings like `"45deg"`, `"135deg"` convert degrees to radians internally
- **`VOID`** = ClassCAD's null/undefined equivalent
- **`TRUE`/`FALSE`** = ClassCAD boolean literals (use JS `true`/`false` in wrappers)

### Batch Operations

Many APIs accept `param` as either a single object or `Array<object>` for batch creation. When batched, the result is an array matching the input order.

### Transformations

- **`[origin, xDir, yDir]`** — 3-point transformation format
- **4×4 matrix** — `[[1,0,0,tx],[0,1,0,ty],[0,0,1,tz],[0,0,0,1]]`
- Left-handed matrices are **not supported**
- Scaling in 4×4 matrices is **ignored**
- Matrices must be **orthogonal**

---

## Typical Workflow

### Part Modeling (feature-based)

```js
// 1. Create a part
const { result: part } = api.v1.part.create({ name: 'MyPart' })

// 2. Add a work plane (optional, default XY exists)
const { result: wp } = api.v1.part.workPlane({ id: part, ... })

// 3. Create a sketch on the work plane
const { result: sketch } = api.v1.sketch.create({ id: part, planeId: wp })

// 4. Draw geometry in the sketch
api.v1.sketch.line({ id: sketch, startPos: [0,0,0], endPos: [100,0,0] })
// ... more geometry, constraints, dimensions

// 5. Extrude the sketch
const { result: extr } = api.v1.part.extrusion({ id: part, ... })

// 6. Add features (fillets, chamfers, booleans, patterns, ...)
```

### Assembly Building

```js
// 1. Create root assembly
const { result: asm } = api.v1.assembly.create({ name: 'MainAssembly' })

// 2. Create or load part templates
const { result: pt } = api.v1.assembly.partTemplate({ name: 'Bracket' })
// ... model the part using api.v1.part.* / api.v1.sketch.*

// 3. Instantiate parts
const { result: inst } = api.v1.assembly.instance({ productId: pt, ownerId: asm })

// 4. Constrain instances
api.v1.assembly.fastened({ id: asm, mate1: {...}, mate2: {...} })
```

### Direct Solid Modeling (non-parametric)

```js
// 1. Create a part and entity injection feature
const { result: part } = api.v1.part.create()
const { result: eif } = api.v1.part.entityInjection({ id: part })

// 2. Create shapes / solids directly
const { result: box } = api.v1.solid.box({ id: eif, length: 100, width: 50, height: 30 })
const { result: cyl } = api.v1.solid.cylinder({ id: eif, height: 50, diameter: 20, translation: [50, 25, 0] })

// 3. Boolean operations
api.v1.solid.subtraction({ id: eif, target: box, tools: [cyl] })
```

### Load / Save

```js
// Load from URL
api.v1.common.load({ url: 'https://example.com/model.ofb', format: 'OFB' })

// Save to data string (base64 encoded)
const { result } = api.v1.common.save({ format: 'OFB', encoding: 'base64' })
// result.content contains the base64 string

// Save as STEP
api.v1.common.save({ file: '/path/to/model.stp', stp: { version: 2 } })
```

---

## Agent Usage Notes

1. **Always read the specific domain reference** before generating API calls — parameter names and types vary per API.
2. **IDs are opaque** — never hardcode them; always capture from a previous API result.
3. **Order matters** for feature-based modeling — features build on previous features.
4. **Sketch workflow**: create sketch → add geometry → add constraints/dimensions → close feature → use in extrusion/revolve.
5. **Entity injection** is the gateway for direct solid/curve operations within a part.
6. **Assembly constraints** (fastened, revolute, cylindrical, etc.) position instances relative to each other using **mates** that reference work coordinate systems (WCS).
7. **Expressions** allow parametric linking: `api.v1.part.expression(...)` + `api.v1.part.linkWithExpression(...)`.
8. **`api.v1.common.batch()`** allows bundling multiple API calls into a single round-trip.
