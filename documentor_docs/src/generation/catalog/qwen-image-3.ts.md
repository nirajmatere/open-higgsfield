# Technical Documentation: `src/generation/catalog/qwen-image-3.ts`

## Overview

The `src/generation/catalog/qwen-image-3.ts` file defines and exports a model configuration for the **Qwen Image 3** model. It uses a helper function (`imageModel`) imported from a local defaults module to construct the model instance with specific identifier, display name, and endpoint mapping properties.

---

## Code Listing

```typescript
import { imageModel } from "./defaults";

export const qwenImage3 = imageModel("qwen-image-3", "Qwen Image 3", {
  text: "alibaba/qwen-image-3/text-to-image",
});
```

---

## Dependencies & Imports

* **`imageModel`** (imported from `./defaults`): A factory function used to initialize and structure an image model entry for the catalog.

---

## Exported Constants

### `qwenImage3`

The `qwenImage3` constant is the default exported configuration object for the Qwen Image 3 model.

#### Parameters Passed to `imageModel`:

1. **Model ID (`"qwen-image-3"`)**:
   * Type: `string`
   * Purpose: Unique internal identifier for the model.

2. **Display Name (`"Qwen Image 3"`)**:
   * Type: `string`
   * Purpose: Human-readable name used to represent the model in UI components or catalog lists.

3. **Endpoints Mapping Object**:
   * Type: `Object`
   * Properties:
     * **`text`** (`"alibaba/qwen-image-3/text-to-image"`): Specifies the provider route/endpoint path used for text-to-image generation requests for this model.

---

## How It Works

1. The file imports the `imageModel` constructor function from `./defaults`.
2. It invokes `imageModel`, passing:
   * The model slug/ID (`"qwen-image-3"`).
   * The human-readable title (`"Qwen Image 3"`).
   * A configuration mapping the `text` modality/task to the target backend string (`"alibaba/qwen-image-3/text-to-image"`).
3. The resulting configured model object is exported via the named export `qwenImage3` for consumption by other parts of the application (such as the model catalog registry).