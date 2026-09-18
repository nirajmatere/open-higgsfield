# Technical Documentation: `src/generation/plane.ts`

## Overview

The `src/generation/plane.ts` module exposes a single utility function, `assemblePlane()`. Its primary purpose is to aggregate state from multiple application stores (`useActive`, `useImagePrompt`, `useVideoPrompt`, `useImageMedia`, `useVideoMedia`, and `useSettings`) along with model metadata from `./catalog` to construct a single `GenerationPlane` data structure.

---

## Dependencies & Imports

- **`getModel`, `parseSettings`** (from `./catalog`): Utility functions used to fetch model configuration metadata and parse model-specific settings.
- **`GenerationPlane`** (from `./catalog/types`): The TypeScript type definition for the object returned by `assemblePlane()`.
- **`useActive`** (from `./stores/active`): State store containing the current active model ID (`model`) and active `surface` ("image" or video).
- **`useImageMedia`, `useVideoMedia`** (from `./stores/media`): State stores containing media items for image and video surfaces, respectively.
- **`useImagePrompt`, `useVideoPrompt`** (from `./stores/prompt`): State stores containing prompt text for image and video surfaces, respectively.
- **`useSettings`** (from `./stores/settings`): State store holding configuration settings grouped by model ID.

---

## Exported Functions

### `assemblePlane(): GenerationPlane`

Reads the current state across relevant stores, applies model role constraints to media items, parses active settings, and returns an assembled `GenerationPlane` object.

#### Internal Execution Flow

1. **Retrieve Active Model & Surface**:
   Reads `model` (aliased as `modelId`) and `surface` from `useActive.getState()`.

2. **Fetch Model Metadata**:
   Calls `getModel(modelId)` to retrieve the model object.

3. **Resolve Prompt Text**:
   Inspects the active `surface`:
   - If `surface === "image"`, retrieves text from `useImagePrompt.getState().text`.
   - Otherwise, retrieves text from `useVideoPrompt.getState().text`.

4. **Resolve Media Items**:
   Inspects the active `surface`:
   - If `surface === "image"`, retrieves media items from `useImageMedia.getState().items`.
   - Otherwise, retrieves media items from `useVideoMedia.getState().items`.

5. **Process and Filter Media by Role Limits**:
   Iterates over the retrieved `items` and groups them into a `media` map indexed by role:
   - Checks `model.roles[item.role]` to determine the maximum allowed items (`max`) for that specific role.
   - If no limit exists (`!max`), the item is skipped.
   - If the accumulated list for that role has already reached `max`, additional items for that role are skipped.
   - Otherwise, the item is added to the role's array in the `media` object.

6. **Parse Settings**:
   Retrieves model-specific settings from `useSettings.getState().byModel[model.id]`. If undefined, defaults to an empty object `{}`. Passes the model and raw settings to `parseSettings(model, ...)`.

7. **Construct and Return `GenerationPlane`**:
   Returns an object with the following shape:

```typescript
{
  model: model.id,
  prompt: { text },
  media,
  settings: parseSettings(model, useSettings.getState().byModel[model.id] ?? {}),
}
```

---

## Behavior Summary Table

| Input State Property | Surface Condition | Source Store / Value |
| :--- | :--- | :--- |
| Active Model ID | N/A | `useActive.getState().model` |
| Active Surface | N/A | `useActive.getState().surface` |
| Prompt Text | `surface === "image"` | `useImagePrompt.getState().text` |
| Prompt Text | `surface !== "image"` | `useVideoPrompt.getState().text` |
| Media Items | `surface === "image"` | `useImageMedia.getState().items` |
| Media Items | `surface !== "image"` | `useVideoMedia.getState().items` |
| Model Settings | N/A | `useSettings.getState().byModel[model.id]` |