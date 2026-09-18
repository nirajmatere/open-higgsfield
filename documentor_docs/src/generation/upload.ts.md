# Technical Documentation: `src/generation/upload.ts`

## Overview

The `src/generation/upload.ts` module provides a single asynchronous utility function, `uploadMedia`, designed to upload a `File` object to Vercel Blob storage. It requests a client token from a backend API endpoint (`/api/blob`) and uses that token to upload the file directly using the `@vercel/blob/client` package.

---

## File Details

* **File Path:** `src/generation/upload.ts`
* **Dependencies:** 
  * `put` from `@vercel/blob/client`

---

## Functions

### `uploadMedia(file: File)`

Uploads a provided `File` to Vercel Blob storage by first requesting an authorization client token from an API route and then directly performing the client-side upload.

#### Function Signature

```typescript
export async function uploadMedia(file: File): Promise<{ url: string }>
```

#### Parameters

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `file` | `File` | The file object to be uploaded to Vercel Blob storage. |

#### Return Value

* **Type:** `Promise<{ url: string }>`
* **Description:** Resolves to an object containing a `url` property, which represents the publicly accessible URL of the uploaded blob.

---

## Step-by-Step Execution Flow

1. **Request Client Token:**
   * Sends a `POST` request to `/api/blob` with the `Content-Type: application/json` header.
   * The JSON body contains the following structure:
     ```json
     {
       "type": "blob.generate-client-token",
       "payload": {
         "pathname": "<file.name>",
         "clientPayload": null,
         "multipart": false
       }
     }
     ```

2. **HTTP Response Validation:**
   * Checks if the response status is successful (`res.ok`).
   * If `res.ok` is `false`, it throws an `Error` with the message: `"Failed to retrieve the client token"`.

3. **Response Payload Parsing & Validation:**
   * Parses the response body as JSON.
   * Checks that both `clientToken` and `pathname` fields are present and of type `string`.
   * If either field fails string type validation, it throws an `Error` with the message: `"Failed to retrieve the client token"`.

4. **Upload File to Blob Storage:**
   * Calls the `put()` function imported from `@vercel/blob/client` passing:
     * `pathname`: The path string received from the API response.
     * `file`: The `File` object passed to `uploadMedia`.
     * Options object: `{ access: "public", token: clientToken }`.

5. **Return Result:**
   * Extracts the `url` from the resulting blob object and returns `{ url: blob.url }`.

---

## Error Handling

The function explicitly throws an `Error("Failed to retrieve the client token")` under the following conditions:

* The `fetch` request to `/api/blob` returns a non-2xx status code (`!res.ok`).
* The parsed JSON response does not contain a string `clientToken`.
* The parsed JSON response does not contain a string `pathname`.