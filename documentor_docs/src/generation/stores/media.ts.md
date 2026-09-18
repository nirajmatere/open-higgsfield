# Technical Documentation: `src/generation/stores/media.ts`

## Overview

The `src/generation/stores/media.ts` module provides client-side state management for handling collections of media items (such as images and videos). Built using [Zustand](https://github.com/pmndrs/zustand) with the `persist` middleware, it exports a reusable factory function and two dedicated stores (`useImageMedia` and `useVideoMedia`) to manage media lists with local storage persistence.

---

## Imported Dependencies

- `create` (`zustand`): Used to build Zustand store instances.
- `persist` (`zustand/middleware`): Middleware that automatically persists store state across page reloads.
- `MediaItem` (`../catalog/types`): The type definition for individual media objects managed by the store.
- `browserStorage` (`./browser-storage`): A custom storage adapter passed to the persistence middleware.

---

## Types

### `MediaState`

Defines the structure of the store's state and available state-mutation methods.

```typescript
type MediaState = {
  items: MediaItem[];
  add: (item: MediaItem) => void;
  remove: (id: string) => void;
};
```

| Property | Type | Description |
| :--- | :--- | :--- |
| `items` | `MediaItem[]` | An array holding the current media items. |
| `add` | `(item: MediaItem) => void` | Appends a new `MediaItem` to the `items` array. |
| `remove` | `(id: string) => void` | Removes a `MediaItem` from the `items` array by its `id`. |

---

## Functions

### `createMediaStore(name: string)`

An internal factory function that generates a Zustand store configured with standard media operations and storage persistence.

#### Parameters
- **`name`** (`string`): Unique key identifier used by the persistence layer to store data under browser storage.

#### Internal State Implementation
- **Initial State**: `items` is initialized to an empty array `[]`.
- **`add(item)`**: Updates `items` by appending the provided `MediaItem` to the existing state array (`[...state.items, item]`).
- **`remove(id)`**: Updates `items` by filtering out any item matching `item.id === id`.

#### Persistence Behavior
The store uses Zustand's `persist` middleware with the following configuration:
- **`name`**: The storage key string passed to `createMediaStore`.
- **`storage`**: Configured using `browserStorage()`.
- **`partialize`**: A filter function that executes prior to writing state to storage. It strips out temporary blob URLs to avoid persisting invalid or unresolvable memory references:
  ```typescript
  partialize: (state) => ({
    items: state.items.filter((item) => !item.url.startsWith("blob:")),
  })
  ```
  *Note: Any item whose `url` begins with `"blob:"` is excluded from browser storage, though it remains in active memory until explicitly removed or the session resets.*

---

## Exported Stores

The file exports two store instances created via `createMediaStore`:

### 1. `useImageMedia`

```typescript
export const useImageMedia = createMediaStore("openhiggsfield.imageMedia.v1");
```
- **Purpose**: Manages state for image media items.
- **Storage Key**: `"openhiggsfield.imageMedia.v1"`

### 2. `useVideoMedia`

```typescript
export const useVideoMedia = createMediaStore("openhiggsfield.videoMedia.v1");
```
- **Purpose**: Manages state for video media items.
- **Storage Key**: `"openhiggsfield.videoMedia.v1"`

---

## Usage Summary

Components can import `useImageMedia` or `useVideoMedia` as standard React hooks to read media items or invoke actions:

```typescript
import { useImageMedia, useVideoMedia } from "src/generation/stores/media";

// Example Usage inside a component
const images = useImageMedia((state) => state.items);
const addImage = useImageMedia((state) => state.add);
const removeImage = useImageMedia((state) => state.remove);
```