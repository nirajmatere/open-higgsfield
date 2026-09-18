# Technical Documentation: `src/generation/catalog/kling-3.ts`

## Overview

The `src/generation/catalog/kling-3.ts` file defines model entries and their associated settings schema for the **Kling 3.0** video generation model catalog. It exports six specific `ModelEntry` objects representing different tiers and capabilities of the Kling 3.0 video generation family.

---

## Dependencies

- **`ModelEntry`** (imported from `./types`): The TypeScript type definition that enforces the required structure for catalog entries and settings objects.

---

## Settings Schema Configurations

The file defines three internal settings objects using `as const satisfies ModelEntry["settings"]`. These objects govern the configurable parameters available for each model variant.

### 1. `klingTurboSettings`
Configuration parameters specific to the **Kling 3.0 Turbo** model variant.

| Setting Parameter | Type | Allowed Values / Range | Default Value |
| :--- | :--- | :--- | :--- |
| `aspectRatio` | `enum` | `"16:9"`, `"9:16"`, `"1:1"` | `"16:9"` |
| `resolution` | `enum` | `"720p"`, `"1080p"` | `"720p"` |
| `duration` | `range` | `min: 3`, `max: 15` | `5` |

### 2. `kling3Settings`
Configuration parameters for standard, professional, and 4K Kling 3.0 models.

| Setting Parameter | Type | Allowed Values / Range | Step | Default Value |
| :--- | :--- | :--- | :--- | :--- |
| `aspectRatio` | `enum` | `"16:9"`, `"9:16"`, `"1:1"` | — | `"16:9"` |
| `duration` | `range` | `min: 3`, `max: 15` | — | `5` |
| `sound` | `boolean` | `true`, `false` | — | `true` |
| `cfgScale` | `range` | `min: 0`, `max: 1` | `0.01` | `0.5` |
| `multiShots` | `boolean` | `true`, `false` | — | `false` |

### 3. `klingMotionSettings`
Configuration parameters for motion control variants of Kling 3.0.

| Setting Parameter | Type | Allowed Values / Range | Default Value |
| :--- | :--- | :--- | :--- |
| `keepOriginalSound` | `boolean` | `true`, `false` | `true` |
| `characterOrientation` | `enum` | `"video"`, `"image"` | `"video"` |

---

## Exported Model Entries

The file exports six `ModelEntry` configurations:

### 1. `kling3Turbo`
* **`id`**: `"kling-3-turbo"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 Turbo"`
* **`roles`**: `{ start: 1 }`
* **`settings`**: References `klingTurboSettings`

### 2. `kling3Std`
* **`id`**: `"kling-3-std"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 Standard"`
* **`roles`**: `{ start: 1, end: 1 }`
* **`settings`**: References `kling3Settings`

### 3. `kling3Pro`
* **`id`**: `"kling-3-pro"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 Pro"`
* **`roles`**: `{ start: 1, end: 1 }`
* **`settings`**: References `kling3Settings`

### 4. `kling34k`
* **`id`**: `"kling-3-4k"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 4K"`
* **`roles`**: `{ start: 1, end: 1 }`
* **`settings`**: References `kling3Settings`

### 5. `kling3MotionStd`
* **`id`**: `"kling-3-motion-std"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 Motion Control"`
* **`roles`**: `{ start: 1, video: 1 }`
* **`settings`**: References `klingMotionSettings`

### 6. `kling3MotionPro`
* **`id`**: `"kling-3-motion-pro"`
* **`surface`**: `"video"`
* **`label`**: `"Kling 3.0 Motion Control Pro"`
* **`roles`**: `{ start: 1, video: 1 }`
* **`settings`**: References `klingMotionSettings`

---

## Summary Table of Models

| Export Constant | Model ID | Surface | Model Label | Associated Roles | Settings Object Used |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `kling3Turbo` | `kling-3-turbo` | `video` | Kling 3.0 Turbo | `start: 1` | `klingTurboSettings` |
| `kling3Std` | `kling-3-std` | `video` | Kling 3.0 Standard | `start: 1, end: 1` | `kling3Settings` |
| `kling3Pro` | `kling-3-pro` | `video` | Kling 3.0 Pro | `start: 1, end: 1` | `kling3Settings` |
| `kling34k` | `kling-3-4k` | `video` | Kling 3.0 4K | `start: 1, end: 1` | `kling3Settings` |
| `kling3MotionStd` | `kling-3-motion-std` | `video` | Kling 3.0 Motion Control | `start: 1, video: 1` | `klingMotionSettings` |
| `kling3MotionPro` | `kling-3-motion-pro` | `video` | Kling 3.0 Motion Control Pro | `start: 1, video: 1` | `klingMotionSettings` |