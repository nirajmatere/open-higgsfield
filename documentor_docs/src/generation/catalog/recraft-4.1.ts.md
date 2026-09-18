# Technical Documentation: `src/generation/catalog/recraft-4.1.ts`

## Overview

The `src/generation/catalog/recraft-4.1.ts` module defines and exports the model configuration for **Recraft 4.1**. It utilizes the `imageModel` helper function imported from `./defaults` to initialize the catalog entry for this specific image model.

---

## File Details

* **File Path:** `src/generation/catalog/recraft-4.1.ts`
* **Primary Export:** `recraft41`

---

## Key Components

### 1. Imports

```typescript
import { imageModel } from "./defaults";
```
* **`imageModel`**: A helper function imported from `./defaults` used to instantiate and configure image model instances for the generation catalog.

### 2. Exports

```typescript
export const recraft41 = imageModel("recraft-4.1", "Recraft 4.1", {
  text: "recraft/v4.1/text-to-image",
});
```

* **`recraft41`**: The exported model configuration object produced by invoking `imageModel`.

---

## Configuration Details

The `imageModel` function is invoked with three arguments:

1. **Identifier (`"recraft-4.1"`):** The unique string ID used internally to reference the Recraft 4.1 model.
2. **Display Name (`"Recraft 4.1"`):** The human-readable name for the model.
3. **Endpoint Mapping Object:**
   * **`text`:** Set to `"recraft/v4.1/text-to-image"`. Specifies the API endpoint route for text-to-image generation requests using Recraft 4.1.

---

## Summary of Functionality

When this module is imported, it executes `imageModel` with the specific parameters for Recraft 4.1 and exports the resulting model object (`recraft41`). This object registers the model's metadata and its text-to-image endpoint path within the application's catalog.