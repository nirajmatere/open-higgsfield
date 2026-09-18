# Technical Documentation: `src/generation/catalog/minimax-hailuo-2.3.ts`

## Overview

The `src/generation/catalog/minimax-hailuo-2.3.ts` module defines and exports a video generation model configuration for **MiniMax Hailuo 2.3**. It leverages helper utility functions from `./defaults` to construct and configure the model definition.

---

## Dependencies

The file imports two helper utilities from the relative module `./defaults`:

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`videoModel`**: A helper function used to construct a video model definition object.
* **`t2v`**: A helper function (short for text-to-video) used to define text-to-video capability for a specific model path or endpoint string.

---

## Exports

### `minimaxHailuo23`

```typescript
export const minimaxHailuo23 = videoModel(
  "minimax-hailuo-2.3",
  "MiniMax Hailuo 2.3",
  { start: 1 },
  t2v("minimax/hailuo-2.3/standard/text-to-video"),
);
```

A named export representing the configured MiniMax Hailuo 2.3 video model instance created by calling `videoModel()`.

---

## Structure & Parameters Breakdown

The `videoModel` function is invoked with four specific parameters:

1. **Identifier (Slug / Key)**: `"minimax-hailuo-2.3"`
   * Unique string identifier for the model.

2. **Display Name**: `"MiniMax Hailuo 2.3"`
   * Human-readable label for the model.

3. **Configuration Object**: `{ start: 1 }`
   * An object specifying operational or configuration properties (sets `start` to `1`).

4. **Task Endpoint / Provider Definition**: `t2v("minimax/hailuo-2.3/standard/text-to-video")`
   * Instantiates a text-to-video task definition pointing to the target path `"minimax/hailuo-2.3/standard/text-to-video"`.