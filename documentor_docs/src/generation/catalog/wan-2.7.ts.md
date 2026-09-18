# Technical Documentation: `src/generation/catalog/wan-2.7.ts`

## Overview

The `src/generation/catalog/wan-2.7.ts` module defines and exports the model configuration for **Wan 2.7**, a text-to-video model entry within the generation catalog. It utilizes builder functions imported from `./defaults` to construct and register the model definition.

---

## Dependencies

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`t2v`**: A helper function imported from `./defaults` used to create a text-to-video configuration or path definition.
* **`videoModel`**: A builder function imported from `./defaults` used to instantiate a video model definition object.

---

## Exported Members

### `wan27`

```typescript
export const wan27 = videoModel("wan-2.7", "Wan 2.7", { start: 1 }, t2v("wan/v2.7/text-to-video"));
```

`wan27` is a named export containing the configured model object returned by `videoModel`.

#### Parameters Passed to `videoModel`:

1. **Model ID / Identifier**: `"wan-2.7"`
   * The unique string key identifying the model.
2. **Display Name**: `"Wan 2.7"`
   * The human-readable name of the model.
3. **Configuration Object**: `{ start: 1 }`
   * An options object defining initial or specific settings for the model (specifying `start: 1`).
4. **Capability Configuration**: `t2v("wan/v2.7/text-to-video")`
   * Specifies the text-to-video resource route or model path `"wan/v2.7/text-to-video"`.

---

## How It Works

1. The file calls `t2v("wan/v2.7/text-to-video")` to generate the text-to-video specification for the model path `"wan/v2.7/text-to-video"`.
2. It passes this specification, along with the model ID (`"wan-2.7"`), display name (`"Wan 2.7"`), and metadata object (`{ start: 1 }`), to the `videoModel` constructor function.
3. The resulting object is stored in the constant `wan27` and exported for use elsewhere in the application catalog.