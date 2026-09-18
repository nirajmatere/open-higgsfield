# Technical Documentation: `src/generation/catalog/pixverse-6.ts`

## Overview

The `src/generation/catalog/pixverse-6.ts` module defines and exports the catalog configuration for the **PixVerse 6** video generation model. It utilizes helper functions from `./defaults` to construct a standardized model definition object for text-to-video generation.

---

## Code Structure & Dependencies

### Imports

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`t2v`**: A helper function imported from `./defaults` that creates a text-to-video task/pipeline configuration targeting a specified endpoint path.
* **`videoModel`**: A factory helper function imported from `./defaults` used to instantiate a video model definition entry.

---

## Exported Artifacts

### `pixverse6`

```typescript
export const pixverse6 = videoModel(
  "pixverse-6", 
  "PixVerse 6", 
  { start: 1 }, 
  t2v("pixverse/v6/text-to-video")
);
```

`pixverse6` is a named export containing the result of calling `videoModel` with specific parameters for the PixVerse 6 model.

#### Parameter Breakdown

1. **Model ID** (`"pixverse-6"`):
   * String identifier for the model.
2. **Display Name** (`"PixVerse 6"`):
   * Human-readable label for the model.
3. **Configuration Object** (`{ start: 1 }`):
   * An options object specifying metadata or constraints (contains `start: 1`).
4. **Task Definition** (`t2v("pixverse/v6/text-to-video")`):
   * The text-to-video pipeline configuration initialized with the API endpoint path `"pixverse/v6/text-to-video"`.

---

## How It Works

1. The module calls `t2v("pixverse/v6/text-to-video")` to initialize the text-to-video task mapping for PixVerse v6.
2. It passes this task configuration, along with the model identifier (`"pixverse-6"`), display name (`"PixVerse 6"`), and options (`{ start: 1 }`), into `videoModel(...)`.
3. The resulting model definition object is exported as `pixverse6` for consumption by the broader model catalog system.