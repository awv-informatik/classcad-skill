# Protocol Envelope

Every ClassCAD API call returns a response envelope. The docs describe it as `{ result, messages?, maxLevel? }` but the actual wire format has **5 keys**:

```js
{
  result:     any,          // the payload — see Result Types below
  messages:   Array,        // always present, always an array (empty [] on success)
  maxLevel:   number,       // highest severity — 31 (info) is the baseline for clean calls
  structure:  object|null,  // full scene graph — ignore for API scripting
  graphic:    object|null,  // rendering data for clients — ignore for API scripting
}
```

## Result Types

| Doc type       | JS type    | Example                        | Notes                                        |
|----------------|------------|--------------------------------|----------------------------------------------|
| `id`           | `number`   | `4`, `52`                      | Positive integers, sequential with gaps      |
| `Array<id>`    | `number[]` | `[58, 64, 70, 76]`            | Array of integer IDs                         |
| `real`         | `number`   | `5`, `0.333`, `3.14159`        | Full JS double precision                     |
| `boolean`      | `number`   | `1`, `0`                       | NOT JS `true`/`false` — always `1` or `0`    |
| `string`       | `string`   | `""`, `"hello"`                | Full Unicode support including emoji          |
| `VOID`         | `null`     | `null`                         | NOT undefined — always `null`                |
| `object`       | `object`   | `{ angleTol: 15, ... }`       | Plain JS object; boolean fields are `1`/`0`  |
| `point`        | `object`   | `{ x: 0, y: 0, z: 0 }`       | Object with x/y/z keys                       |
| `Array<string>`| `string[]` | `["a", "b"]`                   | Plain JS string array, unordered             |

### Type details

**`boolean`:** Universally `1`/`0` numbers — in API results, object fields, and expressions. Never JS `true`/`false`. Expression constants are `TRUE`/`FALSE` (uppercase only; lowercase `true`/`false` are not recognized).

**`id`:** Positive integers. IDs are sequential but with gaps (a part creates ~50 child objects, so the next part ID is ~50 higher). As parameters, IDs accept both numbers and string-encoded numbers (`4` and `"4"` both work). Float, negative, and zero values fail.

**`point`:** Two representations exist — **do not confuse them:**
- **API parameters** use `[x, y, z]` arrays (e.g. `startPos: [0, 0, 0]`)
- **API results, structure tree, and expressions** use `{x, y, z}` objects

**`string`:** Full Unicode including emoji. `getUserData` returns `""` for missing keys (no error) — you cannot distinguish "key exists with empty value" from "key does not exist".

**`VOID`:** All VOID-returning APIs (`clear`, `setObjectName`, `setUserData`, etc.) return `null`.

**Empty arrays:** APIs returning `Array<*>` give `[]` for empty results, not `null`. But on error, result is `null` (not empty array).

**On error:** Result is **always `null`** regardless of declared return type — whether the API normally returns `id`, `boolean`, `Array`, `real`, or `VOID`.

## Error Detection

Check `maxLevel` to detect errors:

```js
const res = await execute({ 'v1.some.api': [{ ... }] })
if (res.maxLevel >= 51) {
  // ERROR — result is likely null, check messages for details
}
```

| maxLevel | Meaning     | Typical scenario                          |
|----------|-------------|-------------------------------------------|
| 31       | info        | Clean success (baseline for all calls)    |
| 41       | warning     | Precursor to error (always paired with 51)|
| 51       | error       | Call failed — result is `null`            |

In practice, `maxLevel == 31` means success, `maxLevel >= 51` means failure. Warning-only (41 without 51) was not observed in testing.

## Messages

```js
{
  message:  string,   // human-readable error/warning text
  level:    number,   // severity: 41=WARNING, 51=ERROR
  levelStr: string,   // UNDOCUMENTED — "WARNING", "ERROR", etc.
  code:     number,   // error code (see table below)
  api:      string,   // INCONSISTENT — sometimes present, sometimes missing
}
```

**Undocumented field:** `levelStr` is always present but not in the API docs.

**Inconsistency:** The `api` field is sometimes omitted (e.g. on `evaluateExpression` errors and unknown command errors).

### Error Codes

| Code | Meaning                          | Example trigger                              |
|------|----------------------------------|----------------------------------------------|
| 0    | General/unclassified             | Internal errors, warnings about ID conversion|
| 1001 | Wrong parameter type             | String where boolean expected                |
| 1004 | Missing required parameter       | Omitting `expression` from evaluateExpression|
| 1006 | Invalid ID                       | Nonexistent ID, string as ID                 |
| 1007 | Wrong ID type                    | Part ID where feature/operation ID expected  |
| 1200 | Root already exists              | Second `part.create` in same drawing         |
| 1201 | Unknown command                  | `v1.common.doesNotExist`, `sketch.isSolved`  |

**Multiple messages per call:** A single call can return multiple messages. Typically a WARNING (41) precedes the ERROR (51) — e.g. "couldn't convert to id" (warning) then "invalid id" (error).

## Batch Envelope

`v1.common.batch` wraps multiple calls. Its envelope structure differs:

**Outer envelope:**
- `result`: `Array` of inner envelopes (one per job)
- `messages`: bubbles up errors from any job, re-attributed to `api: "v1.common.batch"`
- `maxLevel`: highest severity across all jobs

**Inner envelopes (per job):**
- **Success:** `{ result }` only — NO `messages` or `maxLevel` keys
- **Failure:** `{ result, messages, maxLevel }` — full envelope

Batch does NOT stop on error. All jobs execute regardless of earlier failures.

## Parameter Passing

Always pass params as `[{ ... }]` or at minimum `[{}]`/`[]`:

```js
// Good
await execute({ 'v1.common.getAppVersion': [{}] })
await execute({ 'v1.common.getAppVersion': [] })

// Bad — produces broken envelope (result: null, maxLevel: undefined)
await execute({ 'v1.common.getAppVersion': undefined })
```

Extra/unknown parameters are silently ignored — no warning.

## Structure and Graphic Fields

**`structure`:** Full scene graph of the drawing — every object with ID, name, class, parent, children, and members. Updated after every call that modifies the drawing. Massive. Agents should ignore it and use `result` instead.

**`graphic`:** Rendering/tessellation data for client applications. Usually `null` in API scripting contexts.

## Expression Engine

`evaluateExpression` supports a rich expression language:

**Arithmetic:** `+`, `-`, `*`, `/` work on reals. Use `pow(x, y)` for exponentiation — `^` is **NOT** a power operator (silently returns null).

**Functions:** `sin`, `cos`, `sqrt`, `pow`, `exp`, `ln`, etc. Constants use `C:` prefix: `C:PI`.

**Points:** Literal syntax `{x, y, z}` (curly braces, exactly 3 components). Returns `{x, y, z}` object.
- Point arithmetic: `{1,2,3}+{4,5,6}` → `{x:5,y:7,z:9}`
- Scalar multiplication: `{1,2,3}*2` → `{x:2,y:4,z:6}` (commutative)
- `{1,2}` or `{1,2,3,4}` → null (must be exactly 3 components)

**Arrays:** `[1,2,3]` syntax, supports nesting `[[1,2],[3,4]]` and mixed types `[{1,2,3},{4,5,6}]`.

**Booleans:** `TRUE` → `1`, `FALSE` → `0` (uppercase only). Comparison operators (`==`, `>`, `<`) do NOT work.

## Drawing Constraints

- **One root per drawing:** Only one `part.create` or `assembly.create` per drawing. Second call fails with code 1200. Must `clear` first to start over.

## Known Doc Discrepancies

- `v1.sketch.isSolved` — listed in API docs but returns code 1201 (unknown command) on the server.

## Related

- `common.batch` — runs multiple API calls, returns nested envelopes
- `common.evaluateExpression` — useful for testing result types (real, point, VOID)
- `common.getAppVersion` / `common.getClassFileVersion` — simplest stateless calls for connection testing
