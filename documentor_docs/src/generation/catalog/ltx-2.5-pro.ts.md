# Technical Documentation: `src/generation/catalog/ltx-2.5-pro.ts`

## Overview

The `src/generation/catalog/ltx-2.5-pro.ts` file defines and exports a video model configuration constant named `ltx25Pro`. It utilizes helper functions imported from a local defaults module to configure an LTX 2.5 Pro text-to-video model definition.

---

## Imports

The module imports two helpers from `./defaults`:

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`videoModel`**: A helper function used to construct a video model catalog entry.
* **`t2v`**: A helper function used to define text-to-video task parameters or endpoints for a given model.

---

## Exports

### `ltx25Pro`

The module exports a single named constant, `ltx25Pro`, initialized via the `videoModel` function call:

```typescript
export const ltx25Pro = videoModel(
  "ltx-2.5-pro",
  "LTX 2.5 Pro",
  { start: 1 },
  t2v("lightricks/ltx-2.5/text-to-video/pro"),
);
```

#### Parameters Passed to `videoModel`:

1. **Identifier String**: `"ltx-2.5-pro"`  
   The unique string identifier for the model.
2. **Display Name String**: `"LTX 2.5 Pro"`  
   The human-readable name of the model.
3. **Configuration Object**: `{ start: 1 }`  
   An object setting model-specific parameters (specifying `start` set to `1`).
4. **Task Definition**: `t2v("lightricks/ltx-2.5/text-to-video/pro")`  
   The return value of calling the `t2v` helper with the path/identifier string `"lightricks/ltx-2.5/text-to-video/pro"`.

---

## Summary of Execution

When this file is evaluated:
1. It imports `t2v` and `videoModel` from `./defaults`.
2. It invokes `t2v` with `"lightricks/ltx-2.5/text-to-video/pro"`.
3. It passes the resulting value along with `"ltx-2.5-pro"`, `"LTX 2.5 Pro"`, and `{ start: 1 }` into `videoModel`.
4. The result of `videoModel` is exported as `ltx25Pro`.