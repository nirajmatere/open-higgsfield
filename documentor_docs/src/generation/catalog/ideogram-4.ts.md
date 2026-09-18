# Technical Documentation: `src/generation/catalog/ideogram-4.ts`

## Overview

The `src/generation/catalog/ideogram-4.ts` file defines and exports a model configuration for **Ideogram 4.0**. It serves as an entry in the catalog of supported image generation models by instantiating a model definition via a shared utility function.

---

## Key Components

### 1. Module Imports

```typescript
import { imageModel } from "./defaults";
```

* **`imageModel`**: A helper function imported from `./defaults` used to instantiate image model configurations uniformly across the catalog.

---

### 2. Exported Constants

#### `ideogram4`

```typescript
export const ideogram4 = imageModel("ideogram-4", "Ideogram 4.0", {
  text: "ideogram/v4.0",
});
```

The module exports a single constant named `ideogram4`, which holds the object returned by `imageModel`.

---

## Configuration Details

The `imageModel` function is called with three arguments:

1. **ID String (`"ideogram-4"`)**: The internal identifier for this model configuration.
2. **Display Name String (`"Ideogram 4.0"`)**: The human-readable label/title assigned to the model.
3. **Model Mapping Object (`{ text: "ideogram/v4.0" }`)**: A configuration object specifying model identifiers or routes. In this configuration:
   * `text` is set to `"ideogram/v4.0"`.

---

## How It Works

1. Upon module initialization, the file imports the `imageModel` constructor/helper from `./defaults`.
2. It invokes `imageModel` with the identifier `"ideogram-4"`, the display name `"Ideogram 4.0"`, and an object mapping `{ text: "ideogram/v4.0" }`.
3. The resulting model configuration object is exported as `ideogram4`, making it available for import and consumption by other parts of the system requiring access to the Ideogram 4.0 model definitions.