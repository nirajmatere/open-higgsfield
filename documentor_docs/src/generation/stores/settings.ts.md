# Technical Documentation: `src/generation/stores/settings.ts`

## Overview

The `src/generation/stores/settings.ts` file defines a persistent Zustand store (`useSettings`) responsible for managing settings on a per-model basis. It provides state management for model-specific configuration parameters and persists this data to client-side storage using a custom storage provider.

---

## Architecture & Dependencies

### External Dependencies
* **`zustand`**: Used via the `create` function to establish the application state store.
* **`zustand/middleware`**: Provides the `persist` middleware to enable state persistence across page reloads.

### Internal Dependencies
* **`./browser-storage`**: Exports `browserStorage()`, a custom storage wrapper passed to Zustand's persistence middleware.

---

## Type Definitions

### `SettingsState`

Represents the shape of the Zustand store state and its associated actions.

```typescript
type SettingsState = {
  byModel: Record<string, Record<string, unknown>>;
  set: (modelId: string, patch: Record<string, unknown>) => void;
};
```

* **`byModel`**: A dictionary where keys are model IDs (`string`) and values are objects (`Record<string, unknown>`) containing key-value pairs representing settings specific to that model.
* **`set`**: An action method that applies updates (`patch`) to the settings of a specified `modelId`.

---

## Store Configuration (`useSettings`)

The `useSettings` hook is created using `create<SettingsState>()` wrapped with the `persist` middleware.

### Initial State

* **`byModel`**: Defaults to an empty object `{}`.

### Actions & Methods

#### `set(modelId: string, patch: Record<string, unknown>)`

Updates or initializes settings for a specific model identified by `modelId`.

**Logic & Optimization:**
1. Retrieves the current settings for `modelId` via `state.byModel[modelId]`.
2. Performs an equality check on all key-value pairs in the `patch` object against `current` using `Object.is`.
   * **No-op condition**: If `current` exists and every key in `patch` strictly equals the value already stored in `current` (`Object.is(current[key], value)`), the action returns the original `state` object without modification, avoiding unnecessary re-renders.
3. **State Update**: If any value differs, it updates `byModel` by shallow-merging the `current` model settings with the new `patch`:
   ```typescript
   {
     byModel: {
       ...state.byModel,
       [modelId]: { ...current, ...patch },
     }
   }
   ```

---

## Persistence Configuration

The store uses Zustand's `persist` middleware configured with the following options:

| Option | Value | Description |
| :--- | :--- | :--- |
| `name` | `"openhiggsfield.settings.v1"` | The key under which the persisted settings are stored in browser storage. |
| `storage` | `browserStorage()` | Invokes the custom `browserStorage()` helper to handle reading/writing data to local storage. |
| `partialize` | `(state) => ({ byModel: state.byModel })` | A filter function ensuring that only the `byModel` state slice is persisted, excluding action functions like `set`. |

---

## Usage Example

```typescript
import { useSettings } from "./src/generation/stores/settings";

// Accessing state and actions in a component
const settings = useSettings((state) => state.byModel["model-123"]);
const setSettings = useSettings((state) => state.set);

// Updating settings for a specific model
setSettings("model-123", { temperature: 0.7, maxTokens: 100 });
```