# Technical Documentation: `src/generation/to-platform.ts`

## Overview

The `src/generation/to-platform.ts` module is responsible for converting an internal, unified generation payload structure (`GenerationPlane`) into a model- and platform-specific API request specification containing an endpoint `path` and a JSON-compatible request `body`.

It acts as an adapter layer between the application's generic generation representation and specific platform providers (such as Higgsfield AI, Kling Video, and ByteDance Seedance).

---

## Data Structures & Types

### `Mapped`
Represents the target endpoint path and payload body formatted for the underlying API.

```typescript
type Mapped = { 
  path: string; 
  body: Record<string, unknown> 
};
```

* **`path`**: The target API route or model endpoint string.
* **`body`**: Key-value map representing the JSON request body.

### `Mapper`
Function signature for mapping functions.

```typescript
type Mapper = (plane: GenerationPlane) => Mapped;
```

---

## Main Entry Point

### `toPlatform(plane: GenerationPlane): Mapped`

The primary export of the module. Takes a generic `GenerationPlane` object and yields the corresponding `Mapped` object containing the endpoint path and payload.

#### Execution Flow:
1. Resolves the model definition by calling `getModel(plane.model)`.
2. Checks if an explicit mapper exists in the `MAP` object matching `model.id`.
3. If no explicit mapper exists in `MAP`, checks if `model.paths` is defined on the model definition. If present, creates a dynamic mapper using `mapByPaths(next, model.paths)`.
4. Throws an `Error` (`"No platform map for <plane.model>"`) if neither an explicit mapper nor model paths are available.
5. Executes the resolved mapper function with `plane` and returns the `Mapped` result.

---

## Model Mapping Table (`MAP`)

The static `MAP` registry maps explicit model identifier strings (`model.id`) to their dedicated mapping functions:

| Model ID | Handler / Function | Target Endpoint Prefix / Path |
| :--- | :--- | :--- |
| `soul-cinema` | `mapSoul` | `higgsfield-ai/soul/cinema` |
| `soul-2` | `mapSoul` | `higgsfield-ai/soul/v2/standard` |
| `kling-3-turbo` | `mapKlingTurbo` | `kling-video/v3.0-turbo/image-to-video` or `text-to-video` |
| `kling-3-std` | `mapKling3` | `kling-video/v3.0/std/image-to-video` or `text-to-video` |
| `kling-3-pro` | `mapKling3` | `kling-video/v3.0/pro/image-to-video` or `text-to-video` |
| `kling-3-4k` | `mapKling3` | `kling-video/v3.0/4k/image-to-video` or `text-to-video` |
| `kling-3-motion-std` | `mapKlingMotion` | `kling-video/v3/motion-control/std` |
| `kling-3-motion-pro` | `mapKlingMotion` | `kling-video/v3/motion-control/pro` |
| `seedance-2` | `mapSeedance` | `bytedance/seedance-2.0/(image\|reference\|text)-to-video` |
| `seedance-2-fast` | `mapSeedance` | `bytedance/seedance-2.0/fast/(image\|reference\|text)-to-video` |
| `seedance-2-mini` | `mapSeedance` | `bytedance/seedance-2.0/mini/(image\|reference\|text)-to-video` |
| `seedance-2.5` | `mapSeedance` | `bytedance/seedance-2.5/(image\|reference\|text)-to-video` |
| `seedance-2.5-edit` | `mapSeedanceSource` | `bytedance/seedance-2.5/video-edit` |
| `seedance-2.5-extend` | `mapSeedanceSource` | `bytedance/seedance-2.5/video-extend` |

---

## Model-Specific Mapper Functions

### 1. `mapSoul`
Maps parameters for Higgsfield Soul models (`soul-cinema`, `soul-2`).

* **Extracted Payload Fields**:
  * `prompt`: `plane.prompt.text`
  * `batch_size`: `Number(plane.settings.batchSize)`
  * `resolution`: `plane.settings.resolution`
  * `aspect_ratio`: `plane.settings.aspectRatio`
  * `enhance_prompt`: `plane.settings.enhancePrompt`

### 2. `mapKlingTurbo`
Maps requests for the Kling 3.0 Turbo model.

* **Path Selection Logic**:
  * If a `"start"` frame URL is present, path becomes `kling-video/v3.0-turbo/image-to-video`. Payload includes `image_url`.
  * Otherwise, path becomes `kling-video/v3.0-turbo/text-to-video`. Payload includes `aspect_ratio`.
* **Common Payload Fields**: `prompt`, `duration`, `resolution`.

### 3. `mapKling3`
Handles standard Kling 3.0 model variants (`std`, `pro`, `4k`).

* **Path Selection Logic**:
  * If a `"start"` frame URL exists, uses `${prefix}/image-to-video` and sets `image_url` (and optionally `last_image_url` if an `"end"` frame URL exists).
  * Otherwise, uses `${prefix}/text-to-video` and includes `aspect_ratio`.
* **Common Payload Fields**: `prompt`, `sound` (`"on"`/`"off"`), `duration`, `cfg_scale`, `multi_shots`.

### 4. `mapKlingMotion`
Handles Kling 3 motion-control models.

* **Payload Fields**:
  * `prompt`: `plane.prompt.text`
  * `image_url`: Optional (from `"start"` media)
  * `video_url`: Optional (from `"video"` media)
  * `keep_original_sound`: `"yes"` if `plane.settings.keepOriginalSound` is truthy, otherwise `"no"`
  * `character_orientation`: `plane.settings.characterOrientation`

### 5. `mapSeedance`
Handles ByteDance Seedance 2.0 and 2.5 generative generation modes.

* **Path Selection Logic**:
  1. **Image-to-Video**: Triggered if a `"start"` frame URL is present. Uses `${prefix}/image-to-video` with `image_url` and optional `end_image_url`.
  2. **Reference-to-Video**: Triggered if any `"reference"`, `"video"`, or `"audio"` media URLs exist. Uses `${prefix}/reference-to-video` with `aspect_ratio`, `image_urls`, `video_urls`, and `audio_urls`.
  3. **Text-to-Video**: Default fallback. Uses `${prefix}/text-to-video` with `aspect_ratio`.

### 6. `mapSeedanceSource`
Handles ByteDance Seedance 2.5 `video-edit` and `video-extend` routes.

* Extracts first `"video"` media item as `video_url`.
* Maps additional `"video"` items as `video_urls`.
* Maps `"reference"` media as `image_urls`.
* Maps `"audio"` media as `audio_urls`.
* Sets duration based on the `withDuration` boolean flag.

### 7. `mapByPaths`
Fallback mapper utilized when a model relies on standard path configurations specified in `PlatformPaths` rather than explicit custom mappers.

#### Resolution Order / Fallback Rules:
1. **`spec.firstLast` check**: If `spec.firstLast` is defined AND either a `"start"` or `"end"` media URL exists:
   * Returns path `spec.firstLast` with `first_frame_url` and/or `last_frame_url`.
2. **`spec.image` check**: If `spec.image` is defined AND a `"start"` media URL exists:
   * Returns path `spec.image` with `image_url` (and optionally `last_image_url` if an `"end"` media URL exists).
3. **`spec.reference` check**: If `spec.reference` is defined AND reference images or videos exist:
   * Returns path `spec.reference` with `image_urls` and `video_urls`.
4. **`spec.text` check**: If `spec.text` is defined:
   * Returns path `spec.text`. Includes `image_urls` if reference images are present.
5. **Fallback Checks**: If none of the conditions above match, it falls back to return path in order of available specifications: `spec.image`, `spec.reference`, or `spec.firstLast` with the base body payload.
6. **Error Trigger**: If none of `spec.image`, `spec.reference`, or `spec.firstLast` are present, throws an `Error` (`"Model has no platform path"`).

---

## Utility Helper Functions

### `urls(plane: GenerationPlane, role: "start" | "end" | "reference" | "video" | "audio")`
Extracts an array of string URLs from `plane.media[role]`. Returns an empty array `[]` if the role does not exist on `plane.media`.

```typescript
function urls(
  plane: GenerationPlane, 
  role: "start" | "end" | "reference" | "video" | "audio"
): string[]
```

### `seedanceBody(plane: GenerationPlane, withDuration: boolean)`
Constructs common parameters shared across Seedance model payloads.

```typescript
function seedanceBody(plane: GenerationPlane, withDuration: boolean)
```

* **Constructed Payload Fields**:
  * `prompt`: `plane.prompt.text`
  * `resolution`: `plane.settings.resolution`
  * `generate_audio`: `plane.settings.generateAudio`
  * `duration`: Included if `withDuration` is `true` and duration is specified.
  * `output_format`: Included if `plane.settings.outputFormat` is set.

---

## Error Handling

The module raises `Error` exceptions under two specific circumstances:

1. **Missing Mapping Configuration**: Thrown inside `toPlatform()` if a model cannot be resolved to a function in `MAP` and does not have `model.paths` defined:
   ```typescript
   throw new Error(`No platform map for ${plane.model}`);
   ```
2. **Invalid Specification Strategy**: Thrown inside `mapByPaths()` if none of the candidate endpoint options (`text`, `image`, `reference`, `firstLast`) exist on `PlatformPaths`:
   ```typescript
   throw new Error("Model has no platform path");
   ```