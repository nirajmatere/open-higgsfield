# Technical Documentation: `src/openhiggsfield/openhiggsfield-app.tsx`

## Overview

The `src/openhiggsfield/openhiggsfield-app.tsx` file defines the root React component (`OpenHiggsfieldApp`) for the OpenHiggsfield generative AI studio user interface. It manages the complete lifecycle of AI media generation requests (both images and videos), request status polling, persistence of generation history, batch selection and downloads, user feedback/notifications (error states and undo notifications), API credential validation, and deep links into detailed asset viewing.

---

## Key Responsibilities

1. **Generation Lifecycle Management**:
   * Assembles inputs from prompt and settings stores using `assemblePlane()`.
   * Submits requests via `submitGeneration()` and manages placeholder skeleton states during processing.
   * Polls request status via `watchRequest()` and updates history records upon completion or failure.
   * Resumes polling for unfinished runs upon initial application hydration.

2. **History & Persistence**:
   * Hydrates history records asynchronously using `loadHistory()`.
   * Persists history updates back to storage using `saveHistory()`.
   * Supports soft-deleting records with an undo grace period (`UNDO_MS = 6000`).

3. **Gallery & Selection Capabilities**:
   * Filters run records based on active surface (`image` or `video`), `favorites`, or `assets` views.
   * Provides single and multi-item (range-based shift-click) selection.
   * Executes batch operations: favoriting, deleting, and sequential file downloads.

4. **Credential & Error Handling**:
   * Checks for platform credential existence via `hasPlatformCredentials()`.
   * Displays the `KeyModal` when platform credentials are missing.
   * Captures errors during request submission or polling and presents actionable guidance to the user.

---

## Exported Definitions

### Interfaces

#### `ActiveRun`
Represents an active in-flight generation request occupying a tile placeholder in the grid.

```typescript
export interface ActiveRun {
  id: string;          // Identifier for the skeleton tile placeholder
  surface: Surface;    // Media type surface ('image' | 'video')
  modelLabel: string;  // Display label for the generating model
  ratio: string;      // CSS aspect-ratio string
  startedAt: number;   // Epoch timestamp when generation started
}
```

---

## Component API

### `OpenHiggsfieldApp`

The main container component for the OpenHiggsfield application interface.

#### Props
* `fontClassName` (`string`, optional): Optional CSS class string applied to the root container for styling fonts.

```typescript
export function OpenHiggsfieldApp({ fontClassName = "" }: { fontClassName?: string })
```

---

## Internal Types & Helper Functions

### Types

* `RunDraft`: Internal representation of a run before it completes or fails. Contains metadata, settings, prompt, surface type, model details, and timestamps.

### Functions

* `hueOf(seed: string): number`: Computes a deterministic hue value (`0–359`) from a string seed using string character code hashing.
* `rowId(requestId: string, offset: number, count: number): string`: Constructs a unique row ID. Appends `#<offset>` to `requestId` if `count > 1`.
* `draftOf(record: RunRecord): RunDraft`: Extracts a `RunDraft` object from an existing `RunRecord`.
* `runningRows(requestId: string, count: number, draft: RunDraft): RunRecord[]`: Generates an array of initial `RunRecord` items with `status: "running"`.
* `terminalRows(requestId: string, draft: RunDraft, status: GenerationStatus): RunRecord[]`: Constructs final `RunRecord` items with `status: "completed"` or `"failed"` depending on the returned `GenerationStatus`.
* `failedRows(requestId: string, count: number, draft: RunDraft, error: string): RunRecord[]`: Creates `RunRecord` instances explicitly marked with `status: "failed"` and an error message.
* `failureText(status: GenerationStatus): string`: Resolves human-readable error text based on platform status values (`nsfw`, `canceled`, custom string error, or fallback text).
* `describeError(caught: unknown): string`: Formats caught exceptions into user-facing error messages, identifying missing credential errors specifically.

---

## Internal State & Refs

### State Hooks

| State Hook | Type | Description |
| :--- | :--- | :--- |
| `history` | `RunRecord[]` | Array of all generation run records (past and present). |
| `runs` | `ActiveRun[]` | Array of currently active/rendering generation tiles. |
| `error` | `string \| null` | Error message string to display in the composer area. |
| `freshIds` | `string[]` | IDs of completed items recently added to trigger arrival animations. |
| `viewerId` | `string \| null` | ID of the item currently being inspected in the full-screen viewer. |
| `view` | `GalleryView` | Active view filter (`image`, `video`, `assets`, or `favorites`). |
| `focusNonce` | `number` | Incremental counter used to force prompt input refocusing. |
| `historyLoaded`| `boolean` | Flag indicating whether stored history has finished loading. |
| `deleted` | `RunRecord[] \| null` | Holds temporarily deleted records pending the expiration of the undo timer. |
| `selected` | `string[]` | Array of selected record IDs in gallery selection mode. |
| `saving` | `SaveProgress \| null` | Download progress state object during batch asset downloads. |
| `keyConfigured`| `boolean` | True if platform credentials are present. |
| `keysOpen` | `boolean` | Controls visibility of the platform key configuration modal (`KeyModal`). |

### React Refs

| Ref Hook | Initial Value | Purpose |
| :--- | :--- | :--- |
| `galleryRef` | `null` | Ref attached to the scrolling gallery container div. |
| `rangeAnchor` | `null` | Index of the last selected item used for shift-click range selections. |
| `visibleRef` | `[]` | Mirrors the currently visible filtered records without triggering extra re-renders. |
| `press` | `0` | Counter tracking submission clicks to generate unique skeleton tile IDs. |
| `alive` | `true` | Unmount flag preventing async state updates on unmounted component instances. |
| `freshTimers` | `[]` | Array of active timer handles clearing item fresh status after 900ms. |
| `historyRef` | `history` | Ref synchronously holding the latest `history` array. |

---

## Operational Workflows

### 1. Initial Loading & Hydration
* On mount, `loadHistory()` retrieves saved history records and merges them into state using `mergeHistory()`.
* Once hydrated (`historyLoaded` becomes `true`), any items with `status === "running"` trigger `resume()` to re-establish polling watchdogs for active backend requests.
* Credential readiness is checked using `hasPlatformCredentials()`. If credentials are missing, `KeyModal` is displayed automatically.

### 2. Generation Process (`generate`)
1. Validates presence of platform key.
2. Calls `assemblePlane()` to extract current prompt text, surface, model, and active settings.
3. Calculates native batch counts (based on model capabilities or active store batch size).
4. Populates pending `ActiveRun` items into `runs` state to render skeleton placeholder tiles in the UI.
5. Invokes `submitGeneration(plane)` to start backend processing.
6. Writes initial `running` status rows to `history` via `runningRows()`.
7. Calls `resume()` to watch the request until completion or failure.

### 3. Request Watch & Resume (`resume`)
* Calls `watchRequest(requestId, options)` with a poll deadline (`draft.createdAt + POLL_DEADLINE_MS`).
* Upon request completion or failure, updates `history` via `replaceRequest()` and persists changes using `saveHistory()`.
* Newly completed item IDs pass to `markFresh()`, marking them as fresh for 900ms to trigger UI entry transition states.

### 4. Selection & Batch Operations
* Clicking checkboxes toggles single items or ranges (`togglePick`).
* Range selection utilizes `rangeAnchor` and extracts a range slice from `visibleRef.current`.
* **Batch Downloads (`downloadPicked`)**: Iterates sequentially through selected items, downloading files via `saveFile()` while updating `saving` status.
* **Batch Favorites (`favoritePicked`)**: Toggles the favorited state across all selected items.
* **Batch Delete (`deletePicked`)**: Removes selected items from `history` and stores them in `deleted` state.

### 5. Deletion & Undo Flow
* Deleting records places them into the `deleted` state and displays the `UndoBar`.
* An undo timer (`UNDO_MS = 6000`ms) runs. If unhandled before timing out or if closed via `Escape`/dismiss, `deleted` resets to `null`, committing the deletion.
* Triggering `restoreDeleted()` returns items to `history` sorted by `createdAt` timestamp.

---

## Internal Sub-Components

### `UndoBar`

Renders an inline notification bar during the deletion grace period with an animated progress line draining over `6000ms`.

#### Props
* `records`: `RunRecord[]` — Array of deleted records currently held in memory.
* `onUndo`: `() => void` — Callback triggered to restore deleted items.
* `onDismiss`: `() => void` — Callback triggered to dismiss the undo bar and finalize deletion.

```typescript
function UndoBar({
  records,
  onUndo,
  onDismiss,
}: {
  records: RunRecord[];
  onUndo: () => void;
  onDismiss: () => void;
})
```