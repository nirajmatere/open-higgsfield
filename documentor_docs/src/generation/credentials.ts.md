# Technical Documentation: `src/generation/credentials.ts`

## Overview

The `src/generation/credentials.ts` module provides utility functions, constants, and error handling for managing, validating, encoding, and formatting platform API key credentials. It ensures that API keys adhere to a strict `id:secret` format and provides utilities for handling key storage in HTTP cookies and formatting authorization headers.

---

## Constants

### `PLATFORM_KEY_COOKIE`
* **Type:** `string`
* **Value:** `"api_key"`
* **Description:** The key name used when storing the platform API key in cookies.

### `PLATFORM_KEY_COOKIE_OPTIONS`
* **Type:** Object
* **Description:** Configuration options used for setting the cookie storing the platform key.
* **Properties:**
  * `httpOnly` (`boolean`): Set to `true` to prevent client-side JavaScript access.
  * `secure` (`boolean`): Evaluates to `true` if `process.env.NODE_ENV === "production"`, forcing HTTPS transmission in production environments.
  * `sameSite` (`"lax"`): Restricts cross-site cookie transmission to Lax mode.
  * `path` (`string`): Set to `"/"` (available across the entire site).
  * `maxAge` (`number`): `2,592,000` seconds (calculated as `60 * 60 * 24 * 30`, equivalent to 30 days).

---

## Custom Errors

### `MissingCredentialsError`
* **Extends:** `Error`
* **Description:** Error thrown when required platform credentials are not present.
* **Properties:**
  * `message`: `"Missing platform key"`
  * `name`: `"MissingCredentialsError"`

---

## Exported Functions

### `encodeCredentials(apiKey: string): string`
Converts an API key string into a JSON string format.

* **Parameters:**
  * `apiKey` (`string`): The API key to encode.
* **Returns:** `string` — A JSON string formatted as `{"apiKey":"<apiKey>"}`.

---

### `decodeCredentials(raw: string | undefined): { apiKey: string } | null`
Safely parses and validates a raw encoded credential string.

* **Parameters:**
  * `raw` (`string | undefined`): The raw JSON string representing the stored credentials.
* **Returns:** `{ apiKey: string } | null` — An object containing the validated API key, or `null` if parsing fails or validation rules are violated.
* **Behavior & Validation:**
  1. Returns `null` if `raw` is undefined or falsy.
  2. Parses `raw` using `JSON.parse()`.
  3. Validates that the parsed value is a non-null, non-array object.
  4. Extracts the `apiKey` property and verifies it is a non-empty string.
  5. Passes trimmed `apiKey` to `requireIdAndSecret` for format validation.
  6. Returns `null` if any exception is caught or validation fails.

---

### `parseCredentialInput(data: unknown): { apiKey: string }`
Parses and validates arbitrary input data (such as payload or body data) to extract an API key.

* **Parameters:**
  * `data` (`unknown`): The input object to extract credentials from.
* **Returns:** `{ apiKey: string }` — An object containing the validated API key.
* **Throws:**
  * `Error("Enter an API key")` — If `data` is not a valid non-null object, or if neither `apiKey` nor `api_key` properties exist as non-empty strings.
  * `Error("API key must be id:secret")` — Thrown via `requireIdAndSecret` if the API key format is invalid.
* **Behavior:**
  1. Validates `data` is a non-null, non-array object.
  2. Checks for `apiKey` first, falling back to `api_key`.
  3. Trims the string and validates its format using `requireIdAndSecret`.

---

### `toAuthorizationHeader(apiKey: string): string`
Formats an API key for use in an HTTP `Authorization` header.

* **Parameters:**
  * `apiKey` (`string`): The API key to format.
* **Returns:** `string` — The formatted header string (e.g., `"Key <id>:<secret>"`).
* **Throws:** `Error("API key must be id:secret")` — If `apiKey` does not pass `requireIdAndSecret` validation.

---

## Internal Helper Functions

### `requireIdAndSecret(apiKey: string): string` *(Internal)*
Enforces the structure requirement for API keys.

* **Parameters:**
  * `apiKey` (`string`): The API key string to inspect.
* **Returns:** `string` — The validated API key string if valid.
* **Throws:** `Error("API key must be id:secret")` if:
  * The key does not contain a colon (`:`).
  * The colon is at index `0` (missing `id` component).
  * The colon is at the last index (`apiKey.length - 1`, missing `secret` component).