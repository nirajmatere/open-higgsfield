# Technical Documentation: `src/generation/catalog/grok-imagine-2.ts`

## Overview

The `src/generation/catalog/grok-imagine-2.ts` file defines and exports a model configuration object for the **Grok Imagine 2.0** image model. It uses a builder/factory function (`imageModel`) imported from a local defaults module to initialize the model configuration.

---

## Code Breakdown

```typescript
import { imageModel } from "./defaults";

export const grokImagine2 = imageModel("grok-imagine-2", "Grok Imagine 2.0", {
  text: "xai/grok-imagine-image-2.0",
});
```

---

## Dependencies

* **`imageModel`** (imported from `./defaults`): A helper/factory function used to construct an image generation model definition.

---

## Exported Members

### `grokImagine2`

* **Type**: Return type of the `imageModel` function.
* **Export Type**: Named export (`export const grokImagine2`).
* **Description**: Holds the configuration instance for the Grok Imagine 2.0 image model.

---

## Parameters Passed to `imageModel`

The `imageModel` helper function is invoked with three arguments:

1. **Model ID / Identifier**: `"grok-imagine-2"`  
   A unique string identifier for internal catalog reference.

2. **Display Name**: `"Grok Imagine 2.0"`  
   A human-readable label representing the model.

3. **Provider Model Mapping Object**:
   ```typescript
   {
     text: "xai/grok-imagine-image-2.0",
   }
   ```
   * **`text`**: Specifies the underlying provider/model string (`"xai/grok-imagine-image-2.0"`) used for text-to-image generation tasks.