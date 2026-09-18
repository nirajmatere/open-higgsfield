# Technical Documentation: `src/generation/stores/active.ts`

## Overview

The `src/generation/stores/active.ts` file defines a persistent Zustand state store (`useActive`) that manages the currently active configuration for AI content generation. It tracks the selected target surface, the active model ID, and the batch generation count. The store utilizes browser storage for persistence and includes rehydration validation logic to ensure selected models remain valid.

---

## Exports Summary

| Export | Type | Description |
| :--- | :--- | :--- |
| `MAX_BATCH` | `const` (number) | Defines the maximum allowable batch size (`4`). |
| `ActiveState` | `type` | TypeScript interface defining the store's state properties and action signatures. |
| `useActive` | `hook` / `store` | The Zustand store hook for accessing and updating active generation state. |

---

## Detailed Components

### 1. Constants

#### `MAX_BATCH`
```typescript
export const MAX_BATCH = 4;
```
* **Value**: `4`
* **Purpose**: Sets a hard upper boundary for batch generation sizes. Models without native multi-generation support issue individual platform requests per batch unit, so this constant caps the request ceiling.

---

### 2. Type Definitions

#### `ActiveState`
Defines the structure of the store's reactive state and action handlers.

```typescript
type ActiveState = {
  surface: Surface;
  model: string;
  batch: number;
  setModel: (id: string) => void;
  setBatch: (count: number) => void;
};
```

* **Properties**:
  * `surface` (`Surface`): The target generation surface type (e.g., `"video"`), derived from catalog metadata.
  * `model` (`string`): The identifier of the selected model.
  * `batch` (`number`): The current generation count per trigger.
* **Actions**:
  * `setModel`: Function to set a new active model by its ID.
  * `setBatch`: Function to update the batch count.

---

### 3. State Store Implementation (`useActive`)

The `useActive` store is created using Zustand's `create` method and wrapped with the `persist` middleware.

```typescript
export const useActive = create<ActiveState>()(
  persist( ... )
);
```

#### Initial State

| Property | Default Value |
| :--- | :--- |
| `surface` | `"video"` |
| `model` | `"seedance-2.5"` |
| `batch` | `1` |

#### Store Actions

##### `setModel(id: string)`
* **Logic**:
  1. Calls `getModel(id)` from the catalog module (`../catalog`) to retrieve the model object matching the provided `id`.
  2. Compares the returned model's `id` and `surface` against the current store state.
  3. If both match the current state, no state change occurs.
  4. If either differs, updates `model` to `model.id` and `surface` to `model.surface`.

##### `setBatch(count: number)`
* **Logic**:
  1. Rounds the input `count` using `Math.round(count)`.
  2. Clamps the result between `1` and `MAX_BATCH` (`4`) using `Math.min(MAX_BATCH, Math.max(1, ...))`.
  3. If the computed batch count equals the current `state.batch`, no state change occurs.
  4. Otherwise, updates `batch` to the newly calculated value.

---

### 4. Persistence Configuration

The `persist` middleware is configured with the following options:

```typescript
{
  name: "openhiggsfield.active.v2",
  storage: browserStorage(),
  partialize: (state) => ({ surface: state.surface, model: state.model, batch: state.batch }),
  onRehydrateStorage: () => (state) => { ... }
}
```

* **`name`**: `"openhiggsfield.active.v2"` — Storage key used in browser storage.
* **`storage`**: Uses the custom `browserStorage()` module wrapper.
* **`partialize`**: Restricts persisted state strictly to state properties (`surface`, `model`, `batch`), excluding action functions (`setModel`, `setBatch`).
* **`onRehydrateStorage`**: Rehydration lifecycle hook executing validation upon store initialization:
  * Receives the restored state. If `state` is missing/null, execution stops.
  * Executes `getModel(state.model)` inside a `try/catch` block to verify if the persisted model ID exists in the current catalog.
  * If `getModel` throws an error (indicating the persisted model ID is invalid or missing), the catch block resets the active model to default by invoking `state.setModel("seedance-2.5")`.

---

## Dependencies & Imports

* **`zustand`**: Provides `create` for state store creation.
* **`zustand/middleware`**: Provides `persist` middleware for storage integration.
* **`../catalog`**: Provides `getModel(id)` for model validation and surface lookup.
* **`../catalog/types`**: Exports the `Surface` type definition.
* **`./browser-storage`**: Imports `browserStorage` for handling underlying persistence mechanisms.