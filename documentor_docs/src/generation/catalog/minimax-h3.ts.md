# Technical Documentation: `src/generation/catalog/minimax-h3.ts`

## Overview

The `src/generation/catalog/minimax-h3.ts` file defines and exports a video generation model configuration for the **MiniMax H3** model. It registers the model within the generation catalog by utilizing helper functions imported from the local `./defaults` module.

---

## Code Breakdown

```typescript
import { t2v, videoModel } from "./defaults";

export const minimaxH3 = videoModel("minimax-h3", "MiniMax H3", { start: 1 }, t2v("minimax/h3/text-to-video"));
```

---

## Dependencies

*   **`t2v`** *(from `./defaults`)*: A helper function used to define text-to-video capabilities and routes for a given model endpoint identifier.
*   **`videoModel`** *(from `./defaults`)*: A factory/helper function used to construct a standardized video model configuration object for the catalog.

---

## Exported Constants

### `minimaxH3`

The `minimaxH3` constant is the primary export of this module. It represents the initialized catalog definition for the "MiniMax H3" video model.

#### Parameters Passed to `videoModel`:

1.  **Model Identifier (`"minimax-h3"`)**:
    *   **Type**: `string`
    *   **Description**: The unique programmatic key or slug identifying this specific model.

2.  **Display Name (`"MiniMax H3"`)**:
    *   **Type**: `string`
    *   **Description**: The human-readable name of the model.

3.  **Options Object (`{ start: 1 }`)**:
    *   **Type**: `object`
    *   **Description**: A configuration object containing model-specific options (specifically sets `start` to `1`).

4.  **Task Configuration (`t2v("minimax/h3/text-to-video")`)**:
    *   **Type**: Return type of `t2v()`
    *   **Description**: Specifies the underlying text-to-video task target endpoint (`"minimax/h3/text-to-video"`).

---

## How It Works

1. The file imports the `videoModel` constructor and `t2v` helper from `./defaults`.
2. It invokes `t2v("minimax/h3/text-to-video")` to instantiate the text-to-video capability configured for the `"minimax/h3/text-to-video"` path.
3. It passes the model ID (`"minimax-h3"`), human-readable title (`"MiniMax H3"`), options (`{ start: 1 }`), and the text-to-video capability definition into `videoModel`.
4. The resulting object is exported as `minimaxH3` for use across the application catalog.