# Technical Documentation: `src/generation/actions.ts`

## Overview

The `src/generation/actions.ts` file defines Next.js **Server Actions** (indicated by the `"use server"` directive) responsible for managing platform authentication credentials, submitting media generation requests, and polling the status of multiple generation requests in parallel.

---

## Directives & Environment Requirements

* **`"use server"`**: Declares that all exported functions in this module are Next.js Server Actions executed on the server.
* **Environment Variables**:
  * `HF_API_BASE_URL`: Required by `readCredentials()`. Throws an error if this environment variable is missing.

---

## Exported Server Actions

### `savePlatformCredentials(data: unknown)`
Parses and securely stores an API key in a client cookie.

* **Parameters:**
  * `data` (`unknown`): Raw input containing credential data.
* **Process:**
  1. Calls `parseCredentialInput(data)` to extract and validate `apiKey`.
  2. Encodes the API key using `encodeCredentials(apiKey)`.
  3. Writes the encoded key to the browser cookies under `PLATFORM_KEY_COOKIE` using `PLATFORM_KEY_COOKIE_OPTIONS`.

---

### `clearPlatformCredentials()`
Removes stored platform credentials from the client's cookies.

* **Process:**
  1. Sets the cookie named `PLATFORM_KEY_COOKIE` to an empty string (`""`).
  2. Passes `PLATFORM_KEY_COOKIE_OPTIONS` merged with `{ maxAge: 0 }` to force immediate cookie expiration.

---

### `hasPlatformCredentials()`
Checks whether valid credentials currently exist in the user's cookies.

* **Returns:** `Promise<boolean>` — `true` if stored credentials are present and decoded successfully; `false` otherwise.

---

### `submitGeneration(plane: GenerationPlane)`
Validates parameters, converts a generation request payload into platform format, and submits it to the platform client.

* **Parameters:**
  * `plane` (`GenerationPlane`): Configuration object detailing the model and generation settings.
* **Process:**
  1. Fetches the corresponding model schema via `getModel(plane.model)`.
  2. Sanitizes and parses settings using `parseSettings(model, plane.settings)`.
  3. Formats the parsed configuration into endpoint details using `toPlatform(parsed)`, returning `{ path, body }`.
  4. Retrieves valid credentials using `readCredentials()`.
  5. Initializes a platform client via `createPlatformClient(...)` and invokes `submit(path, body)`.

---

### `getGenerationStatuses(data: unknown)`
Fetches the current execution status for a list of generation requests in parallel.

* **Rationale:** Next.js queues client server action calls sequentially per client. Querying requests individually from the client would cause execution bottlenecks. Handing the fan-out on the server enables parallel status queries within a single round trip.
* **Parameters:**
  * `data` (`unknown`): Payload containing an array of request IDs under `data.requestIds`.
* **Returns:** `Promise<StatusResult[]>` — An array of status items, each structured as:
  * `{ requestId, status }` if successful.
  * `{ requestId, error }` if an error occurs while checking that specific ID.
* **Process:**
  1. Validates input using `parseRequestIds(data)`.
  2. Initializes a platform client using `readCredentials()`.
  3. Uses `Promise.all` to concurrently query `client.status(requestId)` for each parsed ID.

---

## Internal Helper Functions

### `readStoredCredentials()`
* **Returns:** `Promise<Credentials | null>`
* Fetches the `PLATFORM_KEY_COOKIE` from the HTTP cookie jar and passes its value to `decodeCredentials()`.

### `readCredentials()`
* **Returns:** `Promise<Credentials & { baseUrl: string }>`
* Reads stored credentials via `readStoredCredentials()`.
* **Throws:**
  * `MissingCredentialsError` if no valid cookie credentials exist.
  * `Error("Missing HF_API_BASE_URL")` if the environment variable `process.env.HF_API_BASE_URL` is undefined.

### `parseRequestIds(data: unknown)`
* **Returns:** `string[]`
* Asserts that `data` is an object containing a non-empty `requestIds` array filled with non-empty strings.
* **Throws:** `Error` if the object shape or any string ID inside the array is invalid.

### `asObject(data: unknown, message: string)`
* **Returns:** `Record<string, unknown>`
* Validates that `data` is a non-null, non-array object.
* **Throws:** `Error` with the provided `message` if validation fails.

---

## Summary of Module Dependencies

| Module | Imported Entities | Purpose |
| :--- | :--- | :--- |
| `next/headers` | `cookies` | Reads and writes HTTP cookies on the server |
| `./catalog` | `getModel`, `parseSettings` | Retrieves model schemas and parses options |
| `./catalog/types` | `GenerationPlane` | Type definition for generation plane input |
| `./credentials` | `MissingCredentialsError`, `PLATFORM_KEY_COOKIE`, `PLATFORM_KEY_COOKIE_OPTIONS`, `decodeCredentials`, `encodeCredentials`, `parseCredentialInput` | Cookie constants and credential parsing/encoding utils |
| `./platform` | `createPlatformClient`, `StatusResult` | API client instantiation and response types |
| `./to-platform` | `toPlatform` | Translates generation configurations into endpoint payloads |