# Technical Documentation: `src/generation/platform.ts`

## Overview

The `src/generation/platform.ts` module provides an HTTP client and validation utilities for interacting with a generation platform service. It manages task submission, generation status retrieval, request payload normalization, error handling, and model identifier validation.

---

## Constants

### `MODEL_ID`
* **Type:** `RegExp`
* **Pattern:** `/^[a-z0-9][a-z0-9._/-]*$/i`
* **Purpose:** A regular expression used to validate model identifiers. Valid IDs must start with an alphanumeric character (`a-z`, `0-9`) followed by optional alphanumeric characters, dots (`.`), underscores (`_`), forward slashes (`/`), or hyphens (`-`).

---

## Exported Error Classes

### `PlatformError`

Custom error class thrown when API requests fail or client-side validation checks fail.

* **Extends:** `Error`
* **Properties:**
  * `readonly status: number`: The HTTP status code (or client-generated status code, e.g., `400` or `502`).
  * `readonly body: unknown`: The parsed response body or error details.
* **Constructor Parameters:**
  * `status: number`
  * `body: unknown`
* **Behavior:**
  * Sets `this.name` to `"PlatformError"`.
  * Generates its error `message` using `messageFromBody(status, body)`.

---

## Exported Types and Interfaces

### `QueuedGeneration`
Represents the payload returned when a generation request is successfully submitted and queued.

```typescript
export type QueuedGeneration = {
  status: string;
  requestId: string;
  statusUrl: string;
  cancelUrl: string;
};
```

### `GenerationStatus`
Represents the current status and output of a generation request.

```typescript
export type GenerationStatus = {
  status: string;
  requestId: string;
  images?: Array<{ url: string }>;
  video?: { url: string };
  error?: unknown;
};
```

### `StatusResult`
A union type representing the outcome of an individual request within a status poll.

```typescript
export type StatusResult =
  | { requestId: string; status: GenerationStatus }
  | { requestId: string; error: string };
```

### `PlatformClientOptions`
Configuration options required to instantiate a platform client.

```typescript
export type PlatformClientOptions = {
  apiKey: string;
  baseUrl: string;
  fetch?: typeof fetch;
};
```

---

## Exported Functions

### `isModelId(model: string): boolean`

Validates whether a given string is a valid model identifier.

* **Parameters:**
  * `model`: `string` – Model name to validate.
* **Returns:** `boolean` – `true` if the string matches `MODEL_ID` and does not contain `..` (preventing path traversal sequences); `false` otherwise.

---

### `createPlatformClient(options: PlatformClientOptions)`

Factory function that creates and returns an HTTP client instance configured to communicate with the platform backend.

* **Parameters:**
  * `options`: `PlatformClientOptions`
* **Initialization Steps:**
  1. Strips trailing slashes from `options.baseUrl`.
  2. Resolves the `fetch` implementation (`options.fetch` or global `fetch`).
  3. Converts `options.apiKey` into an authorization header using `toAuthorizationHeader(options.apiKey)`.

* **Returned Client Interface:**
  The returned object exposes two asynchronous methods:

  #### `submit(model: string, input: Record<string, unknown>): Promise<QueuedGeneration>`
  Submits a new generation job for a specific model.
  * **Validation:** Checks `isModelId(model)`. Throws a `PlatformError(400, { detail: "Invalid model" })` if validation fails.
  * **HTTP Request:** Performs a `POST` request to `/${model}` containing the JSON-serialized `input` object in the body.
  * **Returns:** A `QueuedGeneration` object via `mapQueued`.

  #### `status(requestId: string): Promise<GenerationStatus>`
  Retrieves the status of a previously submitted generation request.
  * **Validation:** Checks if `requestId` is present. Throws a `PlatformError(400, { detail: "Missing request id" })` if missing.
  * **HTTP Request:** Performs a `GET` request to `/requests/${encodeURIComponent(requestId)}/status`.
  * **Returns:** A `GenerationStatus` object via `mapStatus`.

---

## Internal Helper Functions

### `send(method: "GET" | "POST", path: string, body?: Record<string, unknown>)`
Internal function managed inside `createPlatformClient` to handle HTTP transactions.
1. Formats request URL (`${baseUrl}${path}`).
2. Logs outgoing request information via `console.info`.
3. Sets `Authorization` header and conditionally sets `Content-Type: application/json` if a body is present.
4. Performs request via `fetchImpl`.
5. Parses response payload via `readJson`.
6. Logs response status and body via `console.info`.
7. Throws `PlatformError(response.status, payload)` if `response.ok` is `false`.
8. Returns parsed JSON payload.

### `mapQueued(payload: unknown): QueuedGeneration`
Normalizes raw response payloads into `QueuedGeneration` objects.
* Extracts `request_id`, `status` (defaults to `"queued"`), `status_url` (defaults to `""`), and `cancel_url` (defaults to `""`).
* Throws `PlatformError(502, { detail: "Platform response missing request_id" })` if `request_id` is missing or invalid.

### `mapStatus(payload: unknown): GenerationStatus`
Normalizes raw response payloads into `GenerationStatus` objects.
* Extracts `request_id` (defaults to `""`) and `status` (defaults to `"unknown"`).
* Extracts `images` array if valid objects containing string `url` properties are present.
* Extracts `video` object if a string `url` property is present inside a `video` key.
* Retains raw `error` property if defined on payload.

### `asRecord(value: unknown): Record<string, unknown>`
Safely casts a value to a dictionary record object (`Record<string, unknown>`). Returns an empty object `{}` if the input is `null`, not an object, or is an `Array`.

### `stringField(value: Record<string, unknown>, key: string): string | undefined`
Retrieves a property value from a record object if the property exists and its type is `string`. Returns `undefined` otherwise.

### `readJson(response: Response): Promise<unknown>`
Reads text from a Response stream. 
* Returns `null` if empty.
* Returns parsed JSON object if valid.
* Returns raw text string if JSON parsing fails.

### `messageFromBody(status: number, body: unknown): string`
Extracts error details from response bodies.
* Returns `body.detail` if it exists as a non-empty string.
* Fallbacks to `"Platform request failed (${status})"` if `detail` is missing or invalid.