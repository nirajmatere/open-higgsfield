# Technical Documentation: `src/openhiggsfield/idb.ts`

## Overview

The `src/openhiggsfield/idb.ts` module provides a key-value storage abstraction layer for browser and non-browser environments. It primary uses IndexedDB for persistent browser storage, falls back to an in-memory `Map` when IndexedDB is unavailable (such as in server-side or non-browser environments), and provides access to standard synchronous `localStorage` when supported.

---

## Exported Types

### `Kv`

Represents an asynchronous key-value storage interface.

```typescript
export type Kv = {
  get<T>(key: string): Promise<T | undefined>;
  set(key: string, value: unknown): Promise<void>;
};
```

* **`get<T>(key: string): Promise<T | undefined>`**: Retrieves a value associated with the given key. Returns a `Promise` resolving to the typed value or `undefined` if the key does not exist.
* **`set(key: string, value: unknown): Promise<void>`**: Stores a value under the specified key. Returns a `Promise` that resolves when the operation completes.

---

### `LegacyStore`

Represents a synchronous key-value storage interface matching the browser's `Storage` API (e.g., `localStorage`).

```typescript
export type LegacyStore = {
  getItem(key: string): string | null;
  setItem(key: string, value: string): void;
  removeItem(key: string): void;
};
```

* **`getItem(key: string): string | null`**: Fetches a string value by key.
* **`setItem(key: string, value: string): void`**: Saves a string value under a key.
* **`removeItem(key: string): void`**: Removes a key and its associated value.

---

## Module Constants

The module defines internal configuration constants used for IndexedDB setup:

* `DB_NAME`: `"openhiggsfield"` — The name of the IndexedDB database.
* `STORE`: `"kv"` — The object store name inside the IndexedDB database.
* `VERSION`: `1` — The database version number.

---

## Internal Helper Functions

### `open(): Promise<IDBDatabase>`

Opens a connection to the IndexedDB database `"openhiggsfield"` at version `1`.

* **Behavior**:
  * Calls `indexedDB.open(DB_NAME, VERSION)`.
  * **`onupgradeneeded`**: Checks if the object store `"kv"` exists; if not, creates it via `createObjectStore("kv")`.
  * **`onsuccess`**: Resolves the promise with the `IDBDatabase` instance.
  * **`onerror`**: Rejects the promise with `req.error`.

---

### `request<T>(mode: IDBTransactionMode, run: (store: IDBObjectStore) => IDBRequest<T>): Promise<T>`

Executes a database operation within an IndexedDB transaction.

* **Parameters**:
  * `mode`: The transaction mode (`"readonly"` or `"readwrite"`).
  * `run`: Callback function receiving the `IDBObjectStore` instance and returning an `IDBRequest<T>`.
* **Behavior**:
  1. Calls `open()` to acquire the `IDBDatabase` connection.
  2. Creates a transaction on the `"kv"` object store with the specified `mode`.
  3. Executes the provided `run` callback to generate an `IDBRequest`.
  4. Listens to transaction lifecycle events:
     * **`tx.oncomplete`**: Resolves with `req.result`.
     * **`tx.onabort`**: Rejects with `tx.error ?? req.error`.
     * **`tx.onerror`**: Rejects with `tx.error ?? req.error`.

---

## Exported Functions

### `memoryKv(): Kv`

Creates an in-memory implementation of the `Kv` interface backed by a JavaScript `Map`.

* **Returns**: A `Kv` instance.
* **Internal State**: Instantiates an isolated `Map<string, unknown>()`.
* **Methods**:
  * `get<T>(key)`: Resolves asynchronously with `map.get(key) as T | undefined`.
  * `set(key, value)`: Stores the key-value pair in `map` asynchronously.

---

### `idbKv(): Kv`

Creates an IndexedDB-backed implementation of the `Kv` interface.

* **Returns**: A `Kv` instance.
* **Methods**:
  * `get<T>(key)`: Executes a `"readonly"` transaction using `store.get(key)`. Returns `Promise<T | undefined>`.
  * `set(key, value)`: Executes a `"readwrite"` transaction using `store.put(value, key)`. Returns `Promise<void>`.

---

### `defaultKv(): Kv`

Provides the default storage strategy based on environment capabilities.

* **Returns**: A `Kv` instance.
* **Logic**:
  1. Checks if `indexedDB` is undefined (e.g., Node.js / non-browser environment). If so, returns `memoryKv()`.
  2. In browser environments where `indexedDB` is available, lazily initializes a module-scoped singleton `browser` variable using `idbKv()`.
  3. Returns the `browser` `Kv` instance.

---

### `browserLegacy(): LegacyStore | undefined`

Provides safe access to `localStorage` when available.

* **Returns**: The global `localStorage` instance (which conforms to `LegacyStore`), or `undefined` if `localStorage` is not defined in the current environment.

---

## Technical Summary Matrix

| Function | Storage Backend | Environment Compatibility | Persistence | Singleton Cached? |
| :--- | :--- | :--- | :--- | :--- |
| `memoryKv()` | JavaScript `Map` | All environments | In-memory (transient) | No (new instance created on each call) |
| `idbKv()` | IndexedDB (`openhiggsfield` / `kv`) | Environments with `indexedDB` | Persistent | No |
| `defaultKv()` | IndexedDB or `Map` fallback | Universal | Persistent in browser; Transient in non-browser | Yes (caches `idbKv()` in browser mode) |
| `browserLegacy()` | `localStorage` | Environments with `localStorage` | Persistent | Native `localStorage` instance |