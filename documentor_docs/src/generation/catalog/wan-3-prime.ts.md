# Technical Documentation: `src/generation/catalog/wan-3-prime.ts`

## Overview

The `src/generation/catalog/wan-3-prime.ts` module defines and exports the model configuration for **Wan 3.0 Prime**. It utilizes helper functions imported from `./defaults` to initialize a video generation model instance specifically configured for text-to-video generation using the `"alibaba/wan-3.0-prime/text-to-video"` model path.

---

## Code Listing

```typescript
import { t2v, videoModel } from "./defaults";

export const wan3Prime = videoModel(
  "wan-3-prime",
  "Wan 3.0 Prime",
  { start: 1 },
  t2v("alibaba/wan-3.0-prime/text-to-video"),
);
```

---

## Dependencies & Imports

The file imports two helper utilities from the local `./defaults` module:

* **`t2v`**: A function used to define text-to-video model endpoint configurations.
* **`videoModel`**: A factory/builder function used to create a structured video model object.

---

## Exported Constants

### `wan3Prime`

The primary export of this module is the `wan3Prime` constant. It holds the object returned by calling `videoModel(...)` with the following parameters:

1. **Identifier (`string`)**: `"wan-3-prime"`
   * The internal key/slug representing the model.
2. **Display Name (`string`)**: `"Wan 3.0 Prime"`
   * The human-readable name of the model.
3. **Configuration Object (`object`)**: `{ start: 1 }`
   * An object specifying initial or default settings (containing a `start` property set to `1`).
4. **Text-to-Video Pipeline (`ReturnType<typeof t2v>`)**: `t2v("alibaba/wan-3.0-prime/text-to-video")`
   * The pipeline declaration constructed via `t2v()` using the target model address `"alibaba/wan-3.0-prime/text-to-video"`.

---

## How It Works

1. **Pipeline Initialization**: The module invokes `t2v("alibaba/wan-3.0-prime/text-to-video")` to create a text-to-video model reference for the Alibaba Wan 3.0 Prime endpoint.
2. **Model Definition Construction**: The `videoModel` function aggregates the identifier (`"wan-3-prime"`), display name (`"Wan 3.0 Prime"`), configuration (`{ start: 1 }`), and the text-to-video pipeline output into a single model definition structure.
3. **Export**: The resulting object is assigned to `wan3Prime` and exported for use elsewhere in the application catalog.