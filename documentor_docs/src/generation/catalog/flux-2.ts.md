# Technical Documentation: `src/generation/catalog/flux-2.ts`

## Overview

The `src/generation/catalog/flux-2.ts` module defines and exports a configuration instance for the **Flux 2** image model using a helper function imported from `./defaults`.

---

## Dependencies

* **`imageModel`** (imported from `./defaults`)
  A function used to construct and return an image model definition based on the provided parameters.

---

## Exports

### `flux2`

An exported constant representing the configured `Flux 2` image model object created via the `imageModel` helper function.

```typescript
export const flux2 = imageModel("flux-2", "Flux 2", { text: "flux-2-pro" });
```

#### Parameters Passed to `imageModel`:

1. **Identifier (`"flux-2"`)**: A string defining the unique ID or key for this model.
2. **Display Name (`"Flux 2"`)**: A string defining the human-readable name of the model.
3. **Options Object (`{ text: "flux-2-pro" }`)**: An object specifying underlying model mapping parameters, specifically setting the `text` property to `"flux-2-pro"`.

---

## How It Works

1. **Importing Defaults**: The file imports the `imageModel` instantiation function from the relative path `./defaults`.
2. **Model Definition**: It calls `imageModel` with:
   * The identifier `"flux-2"`
   * The label `"Flux 2"`
   * A configuration object containing `{ text: "flux-2-pro" }`
3. **Export**: The output of the `imageModel` invocation is assigned to `flux2` and exported so other modules in the application can reference the Flux 2 model definition.