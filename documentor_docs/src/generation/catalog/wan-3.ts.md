# Technical Documentation: `src/generation/catalog/wan-3.ts`

## Overview

The `src/generation/catalog/wan-3.ts` file is responsible for defining and exporting a video generation model configuration for **Wan 3.0**. It uses helper functions imported from `./defaults` to construct a standardized model definition object.

---

## Key Components

### Imports

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`t2v`**: A helper function imported from `./defaults` used to configure a text-to-video (t2v) model entry using a model path/identifier.
* **`videoModel`**: A helper function imported from `./defaults` used to construct and return a complete video model object with its metadata and capabilities.

---

### Exported Constants

#### `wan3`

```typescript
export const wan3 = videoModel("wan-3", "Wan 3.0", { start: 1 }, t2v("alibaba/wan-3.0/text-to-video"));
```

The module exports a single named constant, `wan3`, which is created by calling `videoModel` with the following parameters:

1. **Identifier**: `"wan-3"`
   * The internal string identifier for the model.
2. **Display Name**: `"Wan 3.0"`
   * The human-readable name for the model.
3. **Configuration Object**: `{ start: 1 }`
   * An object specifying initial configuration options (setting `start` to `1`).
4. **Text-to-Video Pipeline**: `t2v("alibaba/wan-3.0/text-to-video")`
   * The text-to-video definition initialized with the model string `"alibaba/wan-3.0/text-to-video"`.

---

## How It Works

1. **Import Dependencies**: The module loads the builder utilities `t2v` and `videoModel` from `./defaults`.
2. **Configure Text-to-Video Entry**: It invokes `t2v("alibaba/wan-3.0/text-to-video")` to establish the underlying model path for text-to-video processing.
3. **Assemble Video Model Object**: It calls `videoModel(...)` passing the unique ID (`"wan-3"`), the human-readable label (`"Wan 3.0"`), specific parameters (`{ start: 1 }`), and the configured `t2v` entry.
4. **Export**: The resulting object is exported as `wan3` for consumption elsewhere in the application catalog.