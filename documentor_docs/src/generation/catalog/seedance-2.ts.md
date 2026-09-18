# Technical Documentation: `src/generation/catalog/seedance-2.ts`

## Overview

The `src/generation/catalog/seedance-2.ts` module defines configuration objects (`ModelEntry` definitions) for three variants of the **Seedance 2.0** video generation model:
- **Seedance 2.0** (`seedance2`)
- **Seedance 2.0 Fast** (`seedance2Fast`)
- **Seedance 2.0 Mini** (`seedance2Mini`)

These definitions specify model metadata, input media role allocations, and default/allowed parameter settings (such as duration, aspect ratio, audio generation, and resolution).

---

## Dependencies

- **`ModelEntry`** (imported from `./types`): Type definition ensuring that each model entry conforms to the standard shape expected by the model catalog system.
- **`SEEDANCE_ASPECT`** (imported from `./tokens`): An array or set of aspect ratio values accepted by Seedance models.

---

## Internal Shared Constants

To avoid repetition, two shared objects configure standard constraints and settings across all Seedance 2.0 model variants.

### 1. `seedanceRoles`
Defines the numerical capacity or limit for input assets assigned to specific roles during generation:

| Role | Count | Description |
| :--- | :--- | :--- |
| `start` | `1` | Start frame input |
| `end` | `1` | End frame input |
| `reference` | `9` | Reference inputs |
| `video` | `3` | Video inputs |
| `audio` | `3` | Audio inputs |

### 2. `seedanceSettings`
Contains base settings shared across all Seedance 2.0 variants:

- **`aspectRatio`**:
  - `type`: `"enum"`
  - `values`: `SEEDANCE_ASPECT`
  - `default`: `"16:9"`
- **`duration`**:
  - `type`: `"range"`
  - `min`: `4`
  - `max`: `15`
  - `default`: `5`
- **`generateAudio`**:
  - `type`: `"boolean"`
  - `default`: `true`

---

## Exported Model Entries

### 1. `seedance2`

The full standard Seedance 2.0 model entry.

* **`id`**: `"seedance-2"`
* **`surface`**: `"video"`
* **`label`**: `"Seedance 2.0"`
* **`roles`**: `seedanceRoles`
* **`settings`**: Combines `seedanceSettings` with resolution options up to 4K:
  * `aspectRatio`: Enum from `SEEDANCE_ASPECT` (default `"16:9"`)
  * `duration`: Range 4 to 15 (default `5`)
  * `generateAudio`: Boolean (default `true`)
  * `resolution`: Enum `["480p", "720p", "1080p", "4k"]` (default `"720p"`)

---

### 2. `seedance2Fast`

The high-speed variant of the model, restricted to lower resolutions for faster processing.

* **`id`**: `"seedance-2-fast"`
* **`surface`**: `"video"`
* **`label`**: `"Seedance 2.0 Fast"`
* **`roles`**: `seedanceRoles`
* **`settings`**: Combines `seedanceSettings` with restricted resolution options:
  * `aspectRatio`: Enum from `SEEDANCE_ASPECT` (default `"16:9"`)
  * `duration`: Range 4 to 15 (default `5`)
  * `generateAudio`: Boolean (default `true`)
  * `resolution`: Enum `["480p", "720p"]` (default `"720p"`)

---

### 3. `seedance2Mini`

A lightweight variant sharing the same settings constraints as the Fast variant.

* **`id`**: `"seedance-2-mini"`
* **`surface`**: `"video"`
* **`label`**: `"Seedance 2.0 Mini"`
* **`roles`**: `seedanceRoles`
* **`settings`**: Combines `seedanceSettings` with restricted resolution options:
  * `aspectRatio`: Enum from `SEEDANCE_ASPECT` (default `"16:9"`)
  * `duration`: Range 4 to 15 (default `5`)
  * `generateAudio`: Boolean (default `true`)
  * `resolution`: Enum `["480p", "720p"]` (default `"720p"`)

---

## Summary Comparison Table

| Export Name | ID | Resolution Options | Default Resolution | Max Roles (Start / End / Ref / Video / Audio) |
| :--- | :--- | :--- | :--- | :--- |
| `seedance2` | `seedance-2` | `480p`, `720p`, `1080p`, `4k` | `720p` | 1 / 1 / 9 / 3 / 3 |
| `seedance2Fast` | `seedance-2-fast` | `480p`, `720p` | `720p` | 1 / 1 / 9 / 3 / 3 |
| `seedance2Mini` | `seedance-2-mini` | `480p`, `720p` | `720p` | 1 / 1 / 9 / 3 / 3 |