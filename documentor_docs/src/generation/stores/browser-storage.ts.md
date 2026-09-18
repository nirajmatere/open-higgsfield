# Technical Documentation: `src/generation/stores/browser-storage.ts`

## Overview

The `src/generation/stores/browser-storage.ts` module provides a custom `PersistStorage` implementation for Zustand's persistence middleware (`zustand/middleware`). 

Its primary purpose is to provide a safe wrapper around `localStorage` that functions properly in Server-Side Rendering (SSR) environments—where JavaScript modules may evaluate on the server prior to the `window.localStorage` object being defined—as well as handling browser storage exceptions gracefully.

---

## Imports

| Import | Source | Description |
| :--- | :--- | :--- |
| `type PersistStorage` | `zustand/middleware` | TypeScript interface defining the required structure (`getItem`, `setItem`, `removeItem`) for Zustand persistence storage engine implementations. |
| `type StorageValue` | `zustand/middleware` | TypeScript interface representing the wrapped state shape stored by Zustand. |

---

## Exported Functions

### `browserStorage<T>()`

Creates and returns a `PersistStorage<T>` storage object compatible with Zustand.

* **Type Signature**: `<T>() => PersistStorage<T>`
* **Returns**: An object implementing the `PersistStorage<T>` interface containing `getItem`, `setItem`, and `removeItem`.

#### Returned Methods

##### 1. `getItem(name: string)`
* **Purpose**: Retrieves and parses a stored value from `localStorage`.
* **Parameters**:
  * `name` (`string`): The key associated with the stored item.
* **Returns**: `StorageValue<T> | null` (or a Promise resolving to it if required by Zustand's interface, though executed synchronously here).
* **Execution Flow**:
  1. Calls `read(name)` to fetch the raw string value from storage.
  2. If `read` returns `null`, `getItem` returns `null`.
  3. Attempts to parse the raw string using `JSON.parse()`.
  4. If parsing succeeds, returns the parsed object typed as `StorageValue<T>`.
  5. If `JSON.parse()` throws an error, catches the exception and returns `null`.

##### 2. `setItem(name: string, value: StorageValue<T>)`
* **Purpose**: Serializes and writes a value to `localStorage`.
* **Parameters**:
  * `name` (`string`): The key under which to store the item.
  * `value` (`StorageValue<T>`): The state value to be stored.
* **Returns**: `void`
* **Execution Flow**:
  1. Serializes `value` into a JSON string using `JSON.stringify(value)`.
  2. Calls `write(name, jsonString)` to persist the serialized string.

##### 3. `removeItem(name: string)`
* **Purpose**: Deletes an item from `localStorage`.
* **Parameters**:
  * `name` (`string`): The key of the item to remove.
* **Returns**: `void`
* **Execution Flow**:
  1. Calls `write(name, null)`.

---

## Internal Helper Functions

These helper functions are not exported and handle low-level operations and error boundary checks.

### `read(name: string)`

Safely reads a key from `localStorage`.

* **Type Signature**: `(name: string) => string | null`
* **Parameters**:
  * `name` (`string`): The target storage key.
* **Returns**: `string | null`
* **Behavior**:
  1. Checks if `localStorage` is undefined (`typeof localStorage === "undefined"`). If so, returns `null`.
  2. Executes `localStorage.getItem(name)` within a `try/catch` block.
  3. Returns the string value if found, or `null` if any runtime error occurs or if the key does not exist.

### `write(name: string, value: string | null)`

Safely writes to or removes a key from `localStorage`.

* **Type Signature**: `(name: string, value: string | null) => void`
* **Parameters**:
  * `name` (`string`): The target storage key.
  * `value` (`string | null`): The string value to write, or `null` to indicate removal.
* **Returns**: `void`
* **Behavior**:
  1. Checks if `localStorage` is undefined (`typeof localStorage === "undefined"`). If so, exits immediately.
  2. If `value === null`, executes `localStorage.removeItem(name)`.
  3. If `value` is a string, executes `localStorage.setItem(name, value)`.
  4. Encapsulates all operations inside a `try/catch` block. Any storage exceptions (e.g., storage quota exceeded, restricted private browsing mode, or denied access permissions) are silently swallowed.

---

## How It Works

1. **Lazy Environment Check (SSR Safety)**: Instead of evaluating `localStorage` access at the module scope level, `browserStorage` checks for `localStorage` dynamically every time a storage method (`getItem`, `setItem`, `removeItem`) is invoked. This prevents SSR crashes during node/server-side evaluation.
2. **Safe Parsing and Serialization**: Converts state objects to JSON strings via `JSON.stringify` on write operations, and parses them via `JSON.parse` on read operations. If parsing fails, it safely falls back to `null`.
3. **Resilient Exception Handling**: Wraps native `localStorage` methods (`getItem`, `setItem`, `removeItem`) in `try...catch` blocks to prevent browser storage restrictions (such as full quota limits or disabled storage in incognito modes) from throwing unhandled exceptions in the application.