# Technical Documentation: `src/generation/device.ts`

## Overview

The `src/generation/device.ts` module provides utilities for managing device identification, device-related HTTP cookies, and blob storage path generation. It includes features for generating cryptographically secure device identifiers, validating and resolving device IDs, configuring HTTP cookie options, and safely formatting pathnames for file storage.

---

## Constants

### `DEVICE_COOKIE`
- **Type**: `string`
- **Value**: `"ohf_device"`
- **Description**: The standard cookie name used to store the device identifier in HTTP requests/responses.

### `DEVICE_COOKIE_OPTIONS`
- **Type**: `Object`
- **Description**: Configuration options for setting the `ohf_device` cookie.
- **Properties**:
  - `httpOnly` (`boolean`): `true` (prevents client-side JavaScript access to the cookie).
  - `secure` (`boolean`): Evaluates to `true` if `process.env.NODE_ENV === "production"`, otherwise `false`.
  - `sameSite` (`"lax"`): Sets the cookie's `SameSite` attribute to `"lax"`.
  - `path` (`string`): `"/"` (accessible across the entire domain).
  - `maxAge` (`number`): `60 * 60 * 24 * 400` (400 days in seconds).

### `DEVICE_ID_RE` (Internal)
- **Type**: `RegExp`
- **Value**: `/^[A-Za-z0-9_-]{16,64}$/`
- **Description**: Regular expression used to validate device identifiers. A valid device ID must consist of 16 to 64 characters, restricted to uppercase letters, lowercase letters, digits, underscores (`_`), and hyphens (`-`).

---

## Functions

### `mintDeviceId()`

Generates a new, cryptographically random hexadecimal device identifier.

- **Parameters**: None
- **Returns**: `string` — A 32-character hexadecimal string representing the generated device ID.
- **Logic**:
  1. Instantiates a `Uint8Array` of 16 bytes.
  2. Fills the array with cryptographically random values using `crypto.getRandomValues()`.
  3. Converts each byte into a 2-digit zero-padded hexadecimal string.
  4. Joins the array into a single string.

---

### `parseDeviceId(raw)`

Validates a raw string input against the device ID format rules.

- **Parameters**:
  - `raw` (`string | undefined`): The raw input string to validate.
- **Returns**: `string | null` — Returns the `raw` string if it passes validation; otherwise, returns `null`.
- **Logic**: Checks if `raw` is defined and matches `DEVICE_ID_RE`.

---

### `resolveDeviceId(raw)`

Resolves an existing device ID or generates a new one if the input is missing or invalid.

- **Parameters**:
  - `raw` (`string | undefined`): The incoming raw device identifier (e.g., from a request cookie).
- **Returns**: `{ deviceId: string; minted: boolean }`
  - `deviceId`: The validated existing ID or the newly generated ID.
  - `minted`: `true` if a new ID was generated; `false` if an existing valid ID was retained.
- **Logic**:
  1. Attempts to parse `raw` using `parseDeviceId(raw)`.
  2. If valid, returns the parsed ID with `minted: false`.
  3. If invalid or `undefined`, calls `mintDeviceId()` and returns the new ID with `minted: true`.

---

### `blobPathname(deviceId, filename)`

Constructs a sanitized blob storage path string formatted as `<deviceId>/<sanitizedFilename>`.

- **Parameters**:
  - `deviceId` (`string`): The device identifier to validate and prepend to the path.
  - `filename` (`string`): The filename to sanitize and append to the path.
- **Returns**: `string` — The formatted path (e.g., `abc123.../sanitized_file.txt`).
- **Throws**: `Error` ("Invalid device id") if `parseDeviceId(deviceId)` returns `null`.
- **Logic**:
  1. Validates `deviceId` using `parseDeviceId`.
  2. Throws an exception if invalid.
  3. Sanitizes `filename` using the internal `sanitizeFilename` helper.
  4. Combines the valid device ID and sanitized filename separated by a forward slash `/`.

---

### `sanitizeFilename(filename)` (Internal)

Cleans and normalizes raw input filenames to ensure safe storage path attributes.

- **Parameters**:
  - `filename` (`string`): The raw filename input.
- **Returns**: `string` — The sanitized filename.
- **Logic**:
  1. Replaces backslashes (`\`) with forward slashes (`/`).
  2. Extracts the basename (the last segment after splitting by `/`). Defaults to `""` if empty.
  3. Replaces all characters that are not alphanumeric, dots (`.`), underscores (`_`), or hyphens (`-`) with an underscore (`_`).
  4. Removes leading dot characters (`.`) to prevent hidden file extensions.
  5. Truncates the resulting string to a maximum length of 180 characters.
  6. If the final string is empty, defaults to `"file"`.