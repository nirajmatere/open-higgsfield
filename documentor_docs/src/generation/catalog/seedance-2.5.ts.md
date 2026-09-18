# Technical Documentation: `src/generation/catalog/seedance-2.5.ts`

## Overview

The `src/generation/catalog/seedance-2.5.ts` file defines catalog entries for the **Seedance 2.5** family of video generation models. It exports three specific `ModelEntry` configurations:

1. `seedance25`: Base video generation model definition.
2. `seedance25Edit`: Configuration for editing existing video content.
3. `seedance25Extend`: Configuration for extending existing video content.

These objects describe model capabilities, available settings (such as resolution, format, aspect ratio, and duration), and allowed input media roles/limits.

---

## Dependencies & Imports

* **`ModelEntry`** (imported from `./types`): Type definition governing the structure of model catalog entries.
* **`SEEDANCE_ASPECT`** (imported from `./tokens`): An array or list of supported aspect ratio values specific to Seedance models.

---

## Internal Shared Configuration

### `seedance25Settings`

A private, constant settings object typed as `ModelEntry["settings"]` using `as const satisfies`. It establishes default settings shared across the Seedance 2.5 variants:

| Setting Key | Type | Allowed Values | Default Value |
| :--- | :--- | :--- | :--- |
| `resolution` | `enum` | `["480p", "720p"]` | `"720p"` |
| `generateAudio` | `boolean` | `true` or `false` | `true` |
| `outputFormat` | `enum` | `["mp4", "mov"]` | `"mp4"` |

---

## Exported Model Configurations

### 1. `seedance25`

The standard Seedance 2.5 model entry for base video generation.

* **ID**: `"seedance-2.5"`
* **Surface**: `"video"`
* **Label**: `"Seedance 2.5"`

#### Media Input Roles
Specifies the supported input asset roles and their maximum allowed counts:
* `start`: `1`
* `end`: `1`
* `reference`: `30`
* `video`: `10`
* `audio`: `10`

#### Settings
Combines `seedance25Settings` with aspect ratio and duration controls:
* `aspectRatio`: Enum using `SEEDANCE_ASPECT`, default `"16:9"`
* `duration`: Range from `4` to `30` seconds, default `5`
* `resolution`: Enum `["480p", "720p"]`, default `"720p"`
* `generateAudio`: Boolean, default `true`
* `outputFormat`: Enum `["mp4", "mov"]`, default `"mp4"`

---

### 2. `seedance25Edit`

The Seedance 2.5 model entry specifically configured for video editing operations.

* **ID**: `"seedance-2.5-edit"`
* **Surface**: `"video"`
* **Label**: `"Seedance 2.5 Edit"`

#### Media Input Roles
* `video`: `1`
* `reference`: `30`
* `audio`: `10`

#### Settings
Uses the shared `seedance25Settings` directly:
* `resolution`: Enum `["480p", "720p"]`, default `"720p"`
* `generateAudio`: Boolean, default `true`
* `outputFormat`: Enum `["mp4", "mov"]`, default `"mp4"`

---

### 3. `seedance25Extend`

The Seedance 2.5 model entry configured for extending video duration.

* **ID**: `"seedance-2.5-extend"`
* **Surface**: `"video"`
* **Label**: `"Seedance 2.5 Extend"`

#### Media Input Roles
* `video`: `1`
* `reference`: `30`
* `audio`: `10`

#### Settings
Combines `seedance25Settings` with duration configuration:
* `duration`: Range from `4` to `30` seconds, default `5`
* `resolution`: Enum `["480p", "720p"]`, default `"720p"`
* `generateAudio`: Boolean, default `true`
* `outputFormat`: Enum `["mp4", "mov"]`, default `"mp4"`