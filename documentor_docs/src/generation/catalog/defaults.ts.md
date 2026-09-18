# Technical Documentation: `src/generation/catalog/defaults.ts`

## Overview

The `src/generation/catalog/defaults.ts` module provides default configuration constants and factory helper functions for constructing model catalog entries (`ModelEntry`) and platform path mappings (`PlatformPaths`). It standardizes the default settings, surface types, aspect ratios, resolutions, and media roles for image and video generation models.

---

## Imported Types

The file imports three type definitions from `./types`:

*   **`MediaRole`**: Represents the role or media attachment type for generation inputs.
*   **`ModelEntry`**: The structural definition for a model catalog entry.
*   **`PlatformPaths`**: Represents platform route or endpoint paths for text-based and image-based modalities.

---

## Constants

### `IMAGE_ASPECT`
* **Type**: `readonly ["auto", "1:1", "4:3", "3:4", "16:9", "9:16"]`
* **Description**: A constant array defining the permitted aspect ratio options for image models.

### `VIDEO_ASPECT`
* **Type**: `readonly ["16:9", "9:16", "1:1"]`
* **Description**: A constant array defining the permitted aspect ratio options for video models.

---

## Exported Helper Functions

### 1. `t2v(path: string): PlatformPaths`

Generates a `PlatformPaths` object based on an input string path.

#### Logic:
* Checks if the provided `path` string ends with `"/text-to-video"`.
* **If false**: Returns an object with only the `text` path property set to `path`:
  ```ts
  { text: path }
  ```
* **If true**: Returns an object containing both `text` and `image` paths, where the `image` path is derived by replacing the trailing `"/text-to-video"` segment with `"/image-to-video"`:
  ```ts
  {
    text: path,
    image: path.replace(/\/text-to-video$/, "/image-to-video")
  }
  ```

#### Parameters:
* `path` (`string`): The base endpoint path.

#### Return Value:
* `PlatformPaths`: An object containing the mapped route path(s).

---

### 2. `imageModel(id: string, label: string, paths: PlatformPaths): ModelEntry`

Constructs a standard `ModelEntry` object configured specifically for image surface models.

#### Parameters:
* `id` (`string`): Unique identifier for the model.
* `label` (`string`): Human-readable name or label for the model.
* `paths` (`PlatformPaths`): Path configurations mapped via `PlatformPaths`.

#### Pre-configured Defaults:
* **`surface`**: `"image"`
* **`roles`**: `{ reference: 8 }`
* **`settings`**:
  * `aspectRatio`:
    * `type`: `"enum"`
    * `values`: `IMAGE_ASPECT` (`["auto", "1:1", "4:3", "3:4", "16:9", "9:16"]`)
    * `default`: `"1:1"`
  * `resolution`:
    * `type`: `"enum"`
    * `values`: `["1k", "2k", "4k"]`
    * `default`: `"1k"`

#### Return Value:
* `ModelEntry`: An image generation model configuration object.

---

### 3. `videoModel(...)`

Constructs a standard `ModelEntry` object configured specifically for video surface models.

#### Signature:
```ts
export function videoModel(
  id: string,
  label: string,
  roles: Partial<Record<MediaRole, number>>,
  paths: PlatformPaths,
): ModelEntry
```

#### Parameters:
* `id` (`string`): Unique identifier for the model.
* `label` (`string`): Human-readable name or label for the model.
* `roles` (`Partial<Record<MediaRole, number>>`): A map defining media role allowances/limits for the video model.
* `paths` (`PlatformPaths`): Path configurations mapped via `PlatformPaths`.

#### Pre-configured Defaults:
* **`surface`**: `"video"`
* **`roles`**: Configured via the passed `roles` parameter.
* **`settings`**:
  * `aspectRatio`:
    * `type`: `"enum"`
    * `values`: `VIDEO_ASPECT` (`["16:9", "9:16", "1:1"]`)
    * `default`: `"16:9"`
  * `resolution`:
    * `type`: `"enum"`
    * `values`: `["720p", "1080p"]`
    * `default`: `"720p"`
  * `duration`:
    * `type`: `"range"`
    * `min`: `4`
    * `max`: `10`
    * `default`: `5`

#### Return Value:
* `ModelEntry`: A video generation model configuration object.