# Technical Documentation: `src/openhiggsfield/media-tray.tsx`

The `src/openhiggsfield/media-tray.tsx` module provides state management and UI components for handling media file attachments (images, video, audio) within generation workflows. It includes a custom React hook (`useMediaTray`) for managing file selection, uploading, and local upload persistence, as well as a visual component (`MediaStrip`) that displays attached media items above the prompt input.

---

## Directives & Module Dependencies

- **Client Directive**: `"use client"` ensures this file is executed on the client side in Next.js.
- **React Utilities**: `useEffect`, `useRef`, `useState`, `ReactNode`.
- **Domain Types**: `MediaItem`, `MediaRole`, `ModelEntry` from `@/generation/catalog`.
- **Media Stores**: `useImageMedia`, `useVideoMedia` from `@/generation/stores/media`.
- **Upload Utility**: `uploadMedia` from `@/generation/upload`.
- **Local Data Helpers**: `ROLE_ACCEPT`, `ROLE_LABELS`, `ROLE_TAGS`, `rolesOf` from `./data`.
- **Icons**: `AudioIcon`, `CloseIcon`, `VideoIcon` from `./icons`.
- **Upload Persistence Helpers**: `kindOfFile`, `loadUploads`, `mergeUploads`, `rememberUpload`, `saveUploads`, `UploadRecord` from `./uploads`.

---

## Interfaces

### `MediaTray`

The `MediaTray` interface defines the return object structure of the `useMediaTray` hook.

```typescript
export interface MediaTray {
  roles: MediaRole[];
  items: MediaItem[];
  uploads: UploadRecord[];
  staged: string | null;
  uploading: boolean;
  allFull: boolean;
  input: ReactNode;
  begin: (role: MediaRole) => void;
  apply: (role: MediaRole, urls: string[]) => void;
}
```

#### Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `roles` | `MediaRole[]` | Array of roles supported by the active model. |
| `items` | `MediaItem[]` | Current surface's attached media items. |
| `uploads` | `UploadRecord[]` | History of uploaded files persisted in the browser, ordered newest first. |
| `staged` | `string \| null` | URL of the last successfully uploaded file in the current session. |
| `uploading` | `boolean` | Flag indicating whether an asynchronous file upload is currently in progress. |
| `allFull` | `boolean` | Indicates if all roles supported by the active model have reached or exceeded their item limits. |
| `input` | `ReactNode` | Hidden `<input type="file" />` JSX element managed by the hook. |
| `begin` | `(role: MediaRole) => void` | Sets the file picker's allowed extensions (`accept`) for the specified role and triggers the native open file dialog. |
| `apply` | `(role: MediaRole, urls: string[]) => void` | Replaces attached media items for a specific role with the provided list of URLs. |

---

## Internal Helper Functions

### `useMedia(model: ModelEntry)`

Selects and returns the appropriate media store based on the active model's surface property.

- **Parameters**: `model: ModelEntry`
- **Returns**: Returns `imageMedia` (from `useImageMedia()`) if `model.surface === "image"`, otherwise returns `videoMedia` (from `useVideoMedia()`).

---

## Custom Hooks

### `useMediaTray(model, onError)`

Manages the lifecycle of uploading media files, tracking uploaded history, rendering the hidden file input, and synchronizing media attachments for a given model.

```typescript
export function useMediaTray(
  model: ModelEntry,
  onError: (message: string | null) => void,
): MediaTray
```

#### Parameters
- `model: ModelEntry`: Target model configuration.
- `onError: (message: string | null) => void`: Callback function to set or clear error messages during upload.

#### State & Refs
- `uploading` (`boolean`): State tracking active upload process.
- `uploads` (`UploadRecord[]`): State holding the persisted upload history.
- `staged` (`string | null`): State storing the URL of the most recent uploaded item.
- `uploadsLoaded` (`boolean`): State tracking whether stored uploads have finished loading from storage.
- `roleRef` (`useRef<MediaRole>`): Stores the currently active `MediaRole` (defaults to `"reference"`).
- `inputRef` (`useRef<HTMLInputElement>`): Ref attached to the hidden DOM file input element.

#### Lifecycle & Operations

1. **Upload Persistence Loading & Saving**:
   - An initial `useEffect` runs on mount to call `loadUploads()`. Upon resolution, it merges loaded records with existing state using `mergeUploads` and sets `uploadsLoaded` to `true`. Uses a `live` flag to prevent state updates if unmounted during load.
   - A secondary `useEffect` watches `uploads` and `uploadsLoaded`. Whenever `uploads` changes after initial load, `saveUploads(uploads)` is called.

2. **Capacity Computation (`allFull`)**:
   - Computes active roles via `rolesOf(model)`.
   - Counts existing items in the current media store per role.
   - Sets `allFull` to `true` if `roles.length > 0` and every role's item count is greater than or equal to `model.roles[role]`.

3. **File Handling (`onFile`)**:
   - Clears any previous error via `onError(null)` and sets `uploading` state to `true`.
   - Calls `uploadMedia(file)`.
   - On success:
     - Sets `staged` state to `uploaded.url`.
     - Appends the new record to `uploads` state using `rememberUpload`, assigning a generated `crypto.randomUUID()`, the `url`, file `kind` derived via `kindOfFile(file)`, file `name`, and timestamp `createdAt`.
   - On error:
     - Calls `onError` with a contextual error message depending on whether the caught object is an `Error` instance.
   - In `finally`: Sets `uploading` state to `false`.

4. **File Dialog Trigger (`begin`)**:
   - Receives `role: MediaRole`.
   - Updates `roleRef.current` to the selected role.
   - Dynamically updates the DOM file input's `accept` attribute using `ROLE_ACCEPT[role]`.
   - Triggers native file selection dialog by calling `.click()` on `inputRef.current`.

5. **File Input Element (`input`)**:
   - Hidden `<input type="file" />` element.
   - On change, extracts `event.target.files[0]`, resets `event.target.value` to empty string `""`, and invokes `onFile`.

6. **Role Media Synchronization (`apply`)**:
   - Receives target `role` and an array of target `urls`.
   - Identifies items currently assigned to `role` in `media.items`.
   - If an existing item's URL is not in `urls`, removes it via `media.remove(item.id)`.
   - For URLs in `urls` that are not currently in the store for this role, adds them via `media.add({ id: crypto.randomUUID(), url, role })`.

---

## Components

### `MediaStrip`

A React rendering component that displays a horizontal list of attached media items corresponding to active roles configured in the active model.

```typescript
export function MediaStrip({ model }: { model: ModelEntry })
```

#### Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `model` | `ModelEntry` | The active model object defining available roles and surface types. |

#### Behavior & Render Logic

1. Retrieves media state via `useMedia(model)`.
2. Filters `media.items` to only include items whose role exists and is truthy in `model.roles`.
3. If no matching items exist (`items.length === 0`), returns `null`.
4. Renders an unordered list (`<ul className="ohf-strip">`) containing items:
   - For roles `"audio"` or `"video"`, renders icon component `<AudioIcon size={20} />` or `<VideoIcon size={20} />`.
   - For all other roles, renders an standard `<img>` element (`className="ohf-strip-thumb"`). If the image fails to load, the `onError` handler sets `event.currentTarget.style.visibility = "hidden"`.
   - Renders a tag label with `ROLE_TAGS[item.role]`.
   - Renders a removal `<button>` containing `<CloseIcon size={10} />` which calls `media.remove(item.id)` when clicked.