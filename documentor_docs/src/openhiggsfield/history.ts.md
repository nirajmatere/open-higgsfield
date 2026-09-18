# Technical Documentation: `src/openhiggsfield/history.ts`

## Overview

The `src/openhiggsfield/history.ts` module manages the persistence, synchronization, capping, and manipulation of generation run records in the application. It acts as the data layer for tracking image and video generation jobs, supporting dual-tier storage persistence (IndexedDB and legacy Web Storage) with fallback handling for restricted browser environments (e.g., private browsing mode).

---

## Constants

| Constant | Type | Value | Description |
| :--- | :--- | :--- | :--- |
| `HISTORY_KEY` | `string` | `"history.v1"` | Key used for storing history in primary storage (IndexedDB). |
| `LEGACY_HISTORY_KEY` | `string` | `"openhiggsfield.history.v1"` | Key used for storing history in legacy fallback storage (e.g., `localStorage`). |
| `MAX_RECORDS` | `number` | `60` | The default maximum number of unflagged, non-running records retained in history. |

---

## Types & Interfaces

### `RunStatus`

```typescript
export type RunStatus = "running" | "completed" | "failed";
```

Represents the state of a generation job run.

### `RunRecord`

Represents a single entry in the generation history log.

```typescript
export interface RunRecord {
  id: string;
  surface: Surface;
  modelId: string;
  modelLabel: string;
  prompt: string;
  /** CSS aspect-ratio value, e.g. "16 / 9" */
  ratio: string;
  meta: string;
  badge?: string;
  kind: "image" | "video";
  urls: string[];
  status: RunStatus;
  /** Platform request this row is waiting on. Set while status is running so a
      refresh can resume the poll; completed rows keep it for the same id. */
  requestId?: string;
  error?: string;
  /** Layered-gradient fallback used while media loads or when a run failed. */
  art: string;
  createdAt: number;
  /** Kept deliberately: shows in the Favorites scope and outlives the cap. */
  favorite?: boolean;
  /** Resolved catalog settings this run was submitted with, so reuse can
      restore the dials and not just the words. Absent on pre-existing records. */
  settings?: Record<string, unknown>;
}
```

---

## Main API Functions

### Data Persistence

#### `loadHistory`

```typescript
export async function loadHistory(
  kv: Kv = defaultKv(),
  legacy: LegacyStore | undefined = browserLegacy(),
): Promise<RunRecord[]>
```

Reads generation history records from both IndexedDB (`kv`) and legacy storage (`legacy`).
* **Behavior:**
  * Fetches stored records from IndexedDB and legacy storage.
  * If only IndexedDB contains records, returns IndexedDB records.
  * If only legacy storage contains records, returns legacy storage records.
  * If both contain records, returns the merged result via `mergeHistory`.

#### `saveHistory`

```typescript
export async function saveHistory(
  records: RunRecord[],
  kv: Kv = defaultKv(),
  legacy: LegacyStore | undefined = browserLegacy(),
): Promise<void>
```

Validates, caps, and saves the provided array of records into both primary (`kv`) and fallback (`legacy`) stores.
* **Behavior:**
  1. Filters input using `isRunRecord`.
  2. Passes filtered records to `capHistory` to apply record retention limits.
  3. Writes to primary key (`HISTORY_KEY`) and legacy key (`LEGACY_HISTORY_KEY`).
  4. Wraps storage calls in `try...catch` blocks to silently handle browser exceptions (e.g., storage quotas exceeded or private browsing restrictions).

---

### History Manipulation & Lifecycle Management

#### `mergeHistory`

```typescript
export function mergeHistory(stored: RunRecord[], live: RunRecord[]): RunRecord[]
```

Merges stored history records with active session records.
* **Behavior:**
  * Deduplicates records by `id` using a `Map`.
  * Deduplication collisions are resolved via `newerRecord`. Session records take precedence if an in-flight execution landed before IDB finished loading.
  * Sorts records chronologically descending by `createdAt`.
  * Passes the final result through `capHistory`.

#### `capHistory`

```typescript
export function capHistory(records: RunRecord[], max = MAX_RECORDS): RunRecord[]
```

Enforces history retention limits while preserving critical records.
* **Behavior:**
  * Returns original records unchanged if length is less than or equal to `max`.
  * Retention priority rules (records kept regardless of `max` limit):
    * `record.favorite === true` (explicitly saved favorites).
    * `record.status === "running"` (in-flight tasks).
  * Retains up to `max` other standard records before discarding older entries.

#### `replaceRequest`

```typescript
export function replaceRequest(
  records: RunRecord[],
  requestId: string,
  next: RunRecord[],
): RunRecord[]
```

Replaces in-flight (`running`) records matching a specific `requestId` with updated terminal records (`next`).
* **Behavior:**
  * Checks if any running record matches `requestId` (via `requestIdOf`). If none are found (e.g., if deleted by the user), returns `records` unchanged.
  * Filters out records matching `requestId`, inserts `next` records, sorts descending by `createdAt`, and applies `capHistory`.

#### `stepRun`

```typescript
export function stepRun(records: RunRecord[], id: string, delta: number): RunRecord | null
```

Navigates relative to a specific record in the provided history array.
* **Behavior:**
  * Finds index of record with matching `id`.
  * Returns the record located at `from + delta`.
  * Returns `null` if the target ID is not found or if the calculated offset goes past array boundaries (does not wrap around).

---

### Utilities

#### `requestIdOf`

```typescript
export function requestIdOf(record: RunRecord): string
```

Returns the target request identifier for a record.
* **Behavior:** Returns `record.requestId` if present. Fallback behavior parses `record.id` by splitting at the `#` symbol and taking the first segment (`record.id.split("#")[0]!`).

#### `timeAgo`

```typescript
export function timeAgo(timestamp: number, now = Date.now()): string
```

Formats a timestamp into a relative time string.
* **Formatting outputs:**
  * `< 60 seconds`: `"just now"`
  * `< 60 minutes`: `"${minutes} min ago"`
  * `< 24 hours`: `"${hours} hr ago"`
  * `≥ 24 hours`: `"${days} d ago"`

---

## Internal Helper Functions

These internal functions are not exported.

#### `readIdb`
```typescript
async function readIdb(kv: Kv): Promise<RunRecord[]>
```
Fetches raw data from IndexedDB, ensures it is an array, validates each element with `isRunRecord`, and returns capped history. Catches exceptions and returns `[]` on failure.

#### `readLegacy`
```typescript
function readLegacy(legacy: LegacyStore | undefined): RunRecord[]
```
Fetches stringified JSON from legacy storage, parses it, checks for array structure, validates entries with `isRunRecord`, and returns capped history. Catches exceptions and returns `[]` on failure.

#### `newerRecord`
```typescript
function newerRecord(row: RunRecord, prev: RunRecord): boolean
```
Determines precedence when resolving record collisions with identical IDs:
1. Completed or failed records (`status !== "running"`) strictly win over running records.
2. If both records share the same completion state status, the record with the higher or equal `createdAt` timestamp wins.

#### `isRunRecord`
```typescript
function isRunRecord(value: unknown): value is RunRecord
```
Type guard performing strict runtime validation of an unknown object to verify compliance with the `RunRecord` interface structure. Checks correct types for required properties (`id`, `surface`, `prompt`, `ratio`, `urls`, `status`, `createdAt`) and optional properties (`requestId`, `favorite`, `settings`).