# Technical Documentation: `src/generation/stores/prompt.ts`

## Overview

The `src/generation/stores/prompt.ts` module provides Zustand-based state stores for managing text prompts used in image and video generation workflows. It utilizes Zustand's `persist` middleware along with a custom storage implementation (`browserStorage`) to persist prompt text across browser sessions under specific storage keys.

---

## Dependencies

- `zustand`: Used to create React-compatible state stores via the `create` function.
- `zustand/middleware`: Provides the `persist` middleware to save store state to storage.
- `./browser-storage`: Exports `browserStorage`, which acts as the storage adapter for state persistence.

---

## Types

### `PromptState`

Defines the structure of the prompt state store.

```typescript
type PromptState = {
  text: string;
  setText: (text: string) => void;
};
```

#### Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `text` | `string` | Holds the current prompt text value. |
| `setText` | `(text: string) => void` | Updates the `text` value in the store state. |

---

## Functions

### `createPromptStore(name: string)`

A factory function that creates and returns a Zustand store configured with persistence for managing prompt state.

#### Parameters

- `name` (`string`): The key identifier used by the `persist` middleware to save and load state in browser storage.

#### Internal Mechanics

1. **Initial State**: Sets `text` to an empty string (`""`).
2. **State Updates (`setText`)**: Updates the `text` property. It checks if the target `text` is identical to the current `state.text`. If identical, it returns the existing state without triggering a re-render (`state.text === text ? state : { text }`).
3. **Persistence Configuration**:
   - `name`: Uses the supplied `name` parameter.
   - `storage`: Uses the custom `browserStorage()` storage driver.
   - `partialize`: Restricts persisted data to only include `{ text: state.text }`.

---

## Exported Store Hooks

The module exports two store hooks created using `createPromptStore`:

### 1. `useImagePrompt`

A custom React hook / Zustand store for managing image prompt text.

- **Storage Key**: `"openhiggsfield.imagePrompt.v1"`

### 2. `useVideoPrompt`

A custom React hook / Zustand store for managing video prompt text.

- **Storage Key**: `"openhiggsfield.videoPrompt.v1"`

---

## Data Flow & Persistence Summary

1. **State Initialization**: When the store hooks (`useImagePrompt` or `useVideoPrompt`) are instantiated, Zustand restores saved prompt text from `browserStorage()` using their respective keys (`openhiggsfield.imagePrompt.v1` or `openhiggsfield.videoPrompt.v1`).
2. **State Mutation**: Calling `setText(newText)` updates the `text` state property.
3. **Optimized Updates**: If `newText` is equal to the existing `text`, the store state reference is returned as-is to avoid unnecessary state propagation.
4. **Selective Sync**: Upon mutation, the `partialize` configuration isolates the `text` field and updates `browserStorage()`.