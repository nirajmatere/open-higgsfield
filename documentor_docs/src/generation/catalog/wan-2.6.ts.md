# Technical Documentation: `src/generation/catalog/wan-2.6.ts`

## Overview

The `src/generation/catalog/wan-2.6.ts` module defines and exports a model configuration entry for the **Wan 2.6** video generation model. It registers the model's identifier, human-readable display name, initialization settings, and its text-to-video path using helper functions imported from the local catalog defaults.

---

## Dependencies

The module imports two helper utilities from `./defaults`:

```typescript
import { t2v, videoModel } from "./defaults";
```

* **`t2v`**: A utility function used to configure text-to-video settings or endpoints for the model catalog.
* **`videoModel`**: A builder function used to construct a video model catalog object.

---

## Exports

### `wan26`

```typescript
export const wan26 = videoModel("wan-2.6", "Wan 2.6", { start: 1 }, t2v("wan/v2.6/text-to-video"));
```

An exported constant containing the constructed model configuration object for Wan 2.6.

#### Configuration Arguments

The `videoModel` function is invoked with four arguments:

1. **Identifier (`"wan-2.6"`)**: The internal string ID representing the Wan 2.6 model.
2. **Display Name (`"Wan 2.6"`)**: The user-facing name for the model.
3. **Options Object (`{ start: 1 }`)**: A configuration object specifying `{ start: 1 }`.
4. **Text-to-Video Configuration (`t2v("wan/v2.6/text-to-video")`)**: The result of invoking `t2v` with the path string `"wan/v2.6/text-to-video"`.

---

## Summary of Execution

When this module is imported:
1. It calls `t2v("wan/v2.6/text-to-video")` to initialize the text-to-video catalog configuration for Wan 2.6.
2. It passes the ID `"wan-2.6"`, name `"Wan 2.6"`, options `{ start: 1 }`, and the output of `t2v` into `videoModel`.
3. It exports the resulting object as `wan26`.