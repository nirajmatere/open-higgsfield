# Technical Documentation: `src/openhiggsfield/download.ts`

## Overview

The `src/openhiggsfield/download.ts` module provides utility functions for fetching cross-origin media files and saving them directly to the user's local file system, as well as generating standardized, human-readable file names for generated outputs based on run records.

---

## Dependencies & Imports

- **`RunRecord`** (imported from `./history`): A type definition representing a generated run object.

---

## Functions

### `saveFile(url: string, name: string): Promise<boolean>`

Fetches a remote file via standard web APIs, creates an inline Object URL blob, and triggers a browser download action programmatically.

#### Purpose & Background
Standard HTML `<a>` tags with a `download` attribute do not enforce custom file naming when pointing to cross-origin URLs (such as media served from a CDN). `saveFile` circumvents this limitation by downloading the file data via `fetch` as a Blob, creating a same-origin Object URL, and triggering the download link on that Blob.

#### Parameters
- **`url`** (`string`): The remote CDN or asset URL to be fetched.
- **`name`** (`string`): The desired target filename for the downloaded asset.

#### Return Value
- **`Promise<boolean>`**: Resolves to `true` if the file was successfully fetched and the download was triggered; resolves to `false` if the network request fails, returns a non-OK HTTP status, or encounters a CORS/runtime exception.

#### Detailed Execution Flow
1. **Fetch Request**: Executes an `async fetch` call to the specified `url` using `{ mode: "cors" }`.
2. **Status Check**: Verifies `response.ok`. If `false`, returns `false`.
3. **Blob Conversion**: Awaits `response.blob()` to retrieve binary data.
4. **Object URL Creation**: Generates a temporary local URL using `URL.createObjectURL(...)`.
5. **DOM Element Generation**:
   - Programmatically creates an `<a>` element (`document.createElement("a")`).
   - Assigns the local Blob Object URL to the `href` attribute.
   - Sets the `download` attribute to the provided `name`.
   - Triggers the download by invoking `link.click()`.
6. **Cleanup**: Schedules `URL.revokeObjectURL(href)` via `setTimeout` to run after 60,000 milliseconds (60 seconds). This delayed revocation ensures compatibility with browsers like Safari that read Blob data asynchronously after the click event occurs.
7. **Error Handling**: Wraps the operation in a `try...catch` block. Any network error or exception causes the function to catch and return `false`.

---

### `fileNameFor(record: RunRecord, index: number): string`

Generates a standardized, descriptive filename for a run output file based on the prompt text, media extension, and item index.

#### Parameters
- **`record`** (`RunRecord`): The record containing metadata about the run, including prompt text, media type (`kind`), and asset URLs (`urls`).
- **`index`** (`number`): The zero-based index of the asset within the run output array.

#### Return Value
- **`string`**: A formatted file name string using the pattern: `openhiggsfield-<slug>-<1-based-index>.<extension>`

#### Detailed Execution Flow
1. **Extension Extraction**:
   - Inspects `record.urls[0]`.
   - Uses the regular expression `/\.([a-z0-9]{2,4})(?:[?#]|$)/i` to extract the file extension from the URL path.
   - Converts the extracted extension to lowercase.
2. **Prompt Slugification**:
   - Converts `record.prompt` to lowercase.
   - Replaces all sequences of non-alphanumeric characters (`[^a-z0-9]+`) with a single hyphen (`-`).
   - Strips leading and trailing hyphens (`/^-+|-+$/g`).
   - Truncates the slug to a maximum length of 44 characters (`.slice(0, 44)`).
   - Removes any trailing hyphens that remain after truncation (`/-+$/`).
   - Falls back to the string `"run"` if the resulting slug is empty.
3. **Default Extension Fallback**:
   - If no valid extension could be extracted from `record.urls[0]`, it falls back based on `record.kind`:
     - If `record.kind === "video"`, uses `"mp4"`.
     - Otherwise, defaults to `"png"`.
4. **Formatting Output**:
   - Constructs and returns the final string using 1-based indexing (`index + 1`).

---

## File Summary

| Function | Input | Output | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **`saveFile`** | `url: string`, `name: string` | `Promise<boolean>` | Downloads cross-origin media via CORS `fetch` and Blob URL, saving it to disk with a custom filename. |
| **`fileNameFor`** | `record: RunRecord`, `index: number` | `string` | Constructs a sanitized, slugified filename incorporating the prompt, item index, and file extension. |