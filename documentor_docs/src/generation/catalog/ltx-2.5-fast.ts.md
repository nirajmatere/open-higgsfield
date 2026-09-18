# Technical Documentation: `src/generation/catalog/ltx-2.5-fast.ts`

## Overview

The `src/generation/catalog/ltx-2.5-fast.ts` module exports a configured video generation model instance named `ltx25Fast`. It leverages helper utilities imported from a relative `./defaults` module to define a specific text-to-video (T2V) model targeting the Lightricks LTX 2.5 Fast pipeline.

---

## Dependencies

The module imports two utility functions from `./defaults`:

* `t2v`: A helper function used to define or wrap a text-to-video model endpoint path/identifier.
* `videoModel`: A factory function used to construct a structured video model configuration object.

---

## Exported Constants

### `ltx25Fast`

An exported constant created by calling `videoModel` with specific model configuration parameters.

```typescript
export const ltx25Fast = videoModel(
  "ltx-2.5-fast",
  "LTX 2.5 Fast",
  { start: 1 },
  t2v("lightricks/ltx-2.5/text-to-video/fast"),
);
```

---

## Configuration Details

The `videoModel` function call is executed with four arguments:

1. **Model ID / Key**: `"ltx-2.5-fast"`
   * The unique string identifier for this model configuration.
2. **Display Name**: `"LTX 2.5 Fast"`
   * The human-readable label or title for the model.
3. **Options / Metadata Object**: `{ start: 1 }`
   * An object specifying initial model parameters or settings (defines `start: 1`).
4. **Model Endpoint / Handler**: `t2v("lightricks/ltx-2.5/text-to-video/fast")`
   * Specifies the underlying text-to-video model path (`"lightricks/ltx-2.5/text-to-video/fast"`) wrapped by the `t2v` helper.