# Technical Documentation: `src/app/api/blob/route.ts`

## Overview

The `src/app/api/blob/route.ts` file implements a Next.js API route handler for processing Vercel Blob client uploads via a `POST` endpoint. It handles token generation for client-side uploads, prefixing blob storage pathnames with a device identifier, enforcing allowed file content types, and managing device cookies for incoming requests.

---

## Environment Variables

This route relies on the following required environment variable:

* `OPEN_HIGGSFIELD_READ_WRITE_TOKEN`: Authentication token used by `@vercel/blob/client` to manage blob operations. If this variable is missing, the endpoint throws an error.

---

## Primary API Handler

### `POST(request: Request): Promise<NextResponse>`

Processes incoming `POST` requests for client upload tasks using Vercel Blob's `handleUpload` utility.

#### Lifecycle & Execution Flow

1. **Request Body Parsing**: Parses the incoming JSON payload into a `HandleUploadBody` type.
2. **Device Resolution**: 
   * Checks if the upload request type is `"blob.generate-client-token"`.
   * If true, executes `readDeviceId()` to resolve or mint a device ID.
   * Otherwise, sets `device` to `null`.
3. **Path Modification**: If a `device` object exists, calls `withDevicePath` to adjust the target blob pathname using the device ID.
4. **Logging**: Logs an informational message `[blob] upload` summarizing the event payload via `summarizeBlobEvent`.
5. **Token Verification**: Ensures `process.env.OPEN_HIGGSFIELD_READ_WRITE_TOKEN` is present.
6. **Vercel Blob Upload Handshake**: Executes `handleUpload`:
   * Passes the modified request `body`, `request`, and `token`.
   * Defines `onBeforeGenerateToken` callback to enforce allowed content types and suffix rules.
7. **Response Construction**:
   * If both the resulting JSON and request body types are `"blob.generate-client-token"`, returns a JSON response explicitly setting the `pathname` property to `body.payload.pathname`.
   * Otherwise, returns standard `NextResponse.json(json)`.
8. **Cookie Attachment**: Wraps the response in `withDeviceCookie()` to ensure newly minted device IDs are set in response cookies.
9. **Error Handling**:
   * Logs failure details (`[blob] upload failed`).
   * If `device?.minted` is `true`, attaches the device cookie to a `500 Internal Server Error` response rather than failing immediately without setting the cookie.
   * If `device?.minted` is not `true`, re-throws the error.

---

## Allowed Content Types

Configured inside the `onBeforeGenerateToken` callback:

* `image/jpeg`
* `image/png`
* `image/webp`
* `image/gif`
* `video/mp4`
* `audio/wav`
* `audio/x-wav`

`addRandomSuffix` is explicitly set to `true` during token generation.

---

## Helper Functions

### `readDeviceId()`
* **Returns**: `Promise<{ deviceId: string; minted: boolean }>` (resolved via `resolveDeviceId`).
* Reads the cookie value associated with `DEVICE_COOKIE` from the request cookie jar (`cookies()`) and resolves it using `resolveDeviceId`.

### `withDeviceCookie(response: NextResponse, device: { deviceId: string; minted: boolean } | null)`
* **Parameters**:
  * `response`: `NextResponse` instance.
  * `device`: Resolved device object or `null`.
* **Behavior**: If `device` exists and `device.minted` is `true`, sets `DEVICE_COOKIE` on the response using `DEVICE_COOKIE_OPTIONS`. Returns the modified `NextResponse`.

### `withDevicePath(body: HandleUploadBody, deviceId: string)`
* **Parameters**:
  * `body`: Original `HandleUploadBody`.
  * `deviceId`: String identifier for the current device.
* **Behavior**: If `body.type` is `"blob.generate-client-token"`, updates `body.payload.pathname` by calling `blobPathname(deviceId, body.payload.pathname)`. Returns the updated body.

### `summarizeBlobEvent(body: HandleUploadBody)`
* **Parameters**: `body` - The `HandleUploadBody` payload.
* **Returns**: An object formatted for logging:
  * For `"blob.generate-client-token"`: `{ type: body.type, pathname: body.payload.pathname }`
  * For other event types: `{ type: body.type, url: body.payload.blob.url }`

---

## Internal Imports Reference

* `@/generation/device`: Supplies cookie configuration and path resolution utilities:
  * `DEVICE_COOKIE`
  * `DEVICE_COOKIE_OPTIONS`
  * `blobPathname`
  * `resolveDeviceId`