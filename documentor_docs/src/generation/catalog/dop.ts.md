# Technical Documentation: `src/generation/catalog/dop.ts`

## Purpose

The `src/generation/catalog/dop.ts` module is responsible for defining and exporting the configuration for the **DoP** video model within the generation catalog. It uses the `videoModel` factory helper function to construct the `dop` model object.

---

## File Overview

* **File Path:** `src/generation/catalog/dop.ts`
* **Module Type:** TypeScript Module

---

## Dependencies

```typescript
import { videoModel } from "./defaults";
```

* **`videoModel`**: A helper function imported from `./defaults` used to construct a video model configuration object.

---

## Exports

### `dop`

```typescript
export const dop = videoModel("dop", "DoP", { start: 1 }, { image: "higgsfield-ai/dop/lite" });
```

The module exports a single named constant, `dop`, which holds the returned value of the `videoModel` function call.

#### Arguments passed to `videoModel`:

1. **Identifier (`"dop"`)**: The internal string identifier for the model.
2. **Display Name (`"DoP"`)**: The human-readable name of the model.
3. **Start Configuration (`{ start: 1 }`)**: An object specifying a start setting initialized to `1`.
4. **Image Configuration (`{ image: "higgsfield-ai/dop/lite" }`)**: An object specifying the target image repository path (`"higgsfield-ai/dop/lite"`).

---

## How It Works

1. The file imports the `videoModel` initialization function from `./defaults`.
2. It executes `videoModel` with the specific parameters for the "DoP" video model (ID: `"dop"`, Name: `"DoP"`, Start Offset/Index: `1`, Image Path: `"higgsfield-ai/dop/lite"`).
3. The resulting video model object is exported as `dop` for use elsewhere in the application catalog.