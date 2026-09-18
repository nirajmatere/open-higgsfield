# Technical Documentation: `src/openhiggsfield/uploads.ts`

## Overview

The `src/openhiggsfield/uploads.ts` module manages the persistence, retrieval, merging, deduplication, and classification of user file uploads within the application. 

It provides dual-layer persistence mechanisms using modern Key-Value storage (e.g., IndexedDB) and fallback legacy storage (e.g., `localStorage`). The module ensures upload records are sanitized, capped at a maximum count (40 records), deduplicated, and sorted chronologically.

---

## Data Interfaces & Constants

### `UploadRecord`

Represents a single uploaded asset retained in persistent storage.

```typescript
export interface UploadRecord {
  id: string;
  url: string;
  kind: AssetKind;
  name: string;
  createdAt: number;
}
```

#### Fields
* `id` (`string`): Unique identifier for the upload record.
* `url` (`string`): The public and permanent URL pointing to the uploaded asset blob.
* `kind` (`AssetKind`): The coarse asset type (`"image"`, `"video"`, or `"audio"`).
* `name` (`string`): The original file name, used for UI display (title/alt text).
* `createdAt` (`number`): Timestamp indicating when the upload record was created.

---

### Key Constants

* **`UPLOADS_KEY`**: `"uploads.v1"` — Key used for storing records in the primary Key-Value store.
* **`LEGACY_UPLOADS_KEY`**: `"openhiggsfield.uploads.v1"` — Key used for storing records in the legacy storage provider.
* **`MAX_RECORDS`** (Internal): `40` — The maximum number of upload records retained across storage operations.

---

## Exported Functions

### `loadUploads`

Loads upload records from both the primary Key-Value store and the legacy store, merging and returning the result.

```typescript
export async function loadUploads(
  kv: Kv = defaultKv(),
  legacy: LegacyStore | undefined = browserLegacy(),
): Promise<UploadRecord[]>
```

#### Parameters
* `kv` (`Kv`, optional): Primary key-value storage implementation. Defaults to `defaultKv()`.
* `legacy` (`LegacyStore | undefined`, optional): Secondary/legacy storage implementation. Defaults to `browserLegacy()`.

#### Behavior
1. Reads records from `kv` via `readIdb`.
2. Reads records from `legacy` via `readLegacy`.
3. If primary storage returns no records, returns the legacy records.
4. If legacy storage returns no records, returns the primary records.
5. If both storage layers contain records, returns the merged result via `mergeUploads`.

---

### `saveUploads`

Sanitizes, truncates, and persists an array of upload records to both the primary Key-Value store and the legacy store.

```typescript
export async function saveUploads(
  records: UploadRecord[],
  kv: Kv = defaultKv(),
  legacy: LegacyStore | undefined = browserLegacy(),
): Promise<void>
```

#### Parameters
* `records` (`UploadRecord[]`): The array of upload records to persist.
* `kv` (`Kv`, optional): Primary key-value storage instance. Defaults to `defaultKv()`.
* `legacy` (`LegacyStore | undefined`, optional): Secondary storage instance. Defaults to `browserLegacy()`.

#### Behavior
1. Coerces and validates each record in `records` using `coerceUpload`.
2. Filters out invalid records (`null`).
3. Slices the record list to the first 40 records (`MAX_RECORDS`).
4. Persists the clean dataset to `kv` under `UPLOADS_KEY`. Silently catches errors (e.g., private browsing restrictions or denied storage access).
5. Persists the JSON-stringified dataset to `legacy` under `LEGACY_UPLOADS_KEY`. Silently catches errors (e.g., storage quota exceeded).

---

### `mergeUploads`

Merges two collections of upload records, resolving conflicts by retaining the record with the most recent `createdAt` timestamp.

```typescript
export function mergeUploads(
  stored: UploadRecord[],
  live: UploadRecord[]
): UploadRecord[]
```

#### Parameters
* `stored` (`UploadRecord[]`): Existing upload records from primary storage.
* `live` (`UploadRecord[]`): Upload records from secondary/live storage.

#### Behavior
1. Iterates through both record lists combined (`[...stored, ...live]`).
2. Deduplicates records based on `id`. If a record with the same `id` exists, it keeps the record with `createdAt >= existing.createdAt`.
3. Sorts merged records descending by `createdAt` (newest records first).
4. Limits the final array to `MAX_RECORDS` (40).

---

### `rememberUpload`

Prepends a new upload record to an existing list of records while removing duplicates matching the same `url`.

```typescript
export function rememberUpload(
  records: UploadRecord[],
  next: UploadRecord,
  max = MAX_RECORDS,
): UploadRecord[]
```

#### Parameters
* `records` (`UploadRecord[]`): Existing list of upload records.
* `next` (`UploadRecord`): The new upload record to add.
* `max` (`number`, optional): Maximum allowed size of the returned record array. Defaults to `MAX_RECORDS` (`40`).

#### Behavior
1. Filters out existing records from `records` that share the exact same `url` as `next`.
2. Prepends `next` to the front of the filtered array (placing the newest upload first).
3. Limits the result to `max` items.

---

### `kindOfFile`

Determines the coarse `AssetKind` category for a given Web API `File` based on its MIME type.

```typescript
export function kindOfFile(file: File): AssetKind
```

#### Parameters
* `file` (`File`): A browser `File` object.

#### Returns
* `"video"` if `file.type` starts with `"video/"`.
* `"audio"` if `file.type` starts with `"audio/"`.
* `"image"` for all other file MIME types (including unrecognised or fallback types).

---

## Internal Helper Functions

### `readIdb`

```typescript
async function readIdb(kv: Kv): Promise<UploadRecord[]>
```
* Asynchronously retrieves data stored at `UPLOADS_KEY` from the provided `Kv` store.
* Verifies that the retrieved value is an array.
* Maps through `coerceUpload`, filters invalid entries, and limits results to 40 items.
* Catches any errors during retrieval and returns an empty array `[]`.

### `readLegacy`

```typescript
function readLegacy(legacy: LegacyStore | undefined): UploadRecord[]
```
* Reads and parses raw JSON string stored at `LEGACY_UPLOADS_KEY` in the `legacy` store.
* Returns an empty array `[]` if `legacy` is undefined, if no item exists, or if parsing fails.
* Validates array structure, coercing and filtering records before truncating to 40 items.

### `coerceUpload`

```typescript
function coerceUpload(value: unknown): UploadRecord | null
```
* Validates runtime unknown data structures into a strictly typed `UploadRecord`.
* Ensures `id`, `url`, and `name` are strings, and `createdAt` is a number. Returns `null` if any validation fails.
* Validates `kind`: if value is `"video"`, `"audio"`, or `"image"`, it is preserved; otherwise, it defaults to `"image"`.