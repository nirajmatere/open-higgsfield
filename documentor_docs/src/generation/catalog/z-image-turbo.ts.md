# Technical Documentation: `src/generation/catalog/z-image-turbo.ts`

## Overview

The `src/generation/catalog/z-image-turbo.ts` file defines and exports a single model configuration constant named `zImageTurbo`. It utilizes a helper function (`imageModel`) imported from a relative `./defaults` module to construct the model configuration.

---

## Dependencies

* **`imageModel`** (imported from `./defaults`): A factory or helper function used to initialize and register an image generation model definition with specific parameters.

---

## Exported Constants

### `zImageTurbo`

`zImageTurbo` is an exported constant created by executing the `imageModel` function with three arguments:

1. **Model Key / ID**: `"z-image-turbo"` — A string identifier for the model.
2. **Model Name**: `"Z-Image Turbo"` — A human-readable display name or title for the model.
3. **Configuration Object**: `{ text: "z-image/turbo" }` — An object specifying additional properties or endpoints associated with this model (specifically mapping `text` to `"z-image/turbo"`).

```typescript
export const zImageTurbo = imageModel("z-image-turbo", "Z-Image Turbo", {
  text: "z-image/turbo",
});
```

---

## How It Works

1. The module imports the `imageModel` constructor/helper function from `./defaults`.
2. It invokes `imageModel`, passing the string identifier `"z-image-turbo"`, the display label `"Z-Image Turbo"`, and a configuration object `{ text: "z-image/turbo" }`.
3. The resulting return value of `imageModel` is assigned to `zImageTurbo` and exported for use in other parts of the application catalog.