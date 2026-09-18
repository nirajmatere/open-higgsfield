# Developer Documentation: `src/generation/catalog/soul.ts`

## Overview

The `src/generation/catalog/soul.ts` file defines model configurations and catalog entries for the "Soul" family of image generation models within the system. It exports two specific model configurations—`soulCinema` and `soul2`—both sharing a common set of generation settings (`soulSettings`).

---

## Dependencies

* **`ModelEntry`** (Imported from `./types`): The TypeScript interface or type that defines the schema for a model catalog entry.
* **`SOUL_ASPECT`** (Imported from `./tokens`): A constant defining the valid aspect ratio values applicable to Soul models.

---

## Internal Configurations

### `soulSettings`

`soulSettings` is an immutable settings object enforced by TypeScript's `as const satisfies ModelEntry["settings"]`. It specifies the configurable parameters, input types, allowed values, and defaults for the Soul models.

| Setting Property | Type | Allowed Values | Default Value | Description |
| :--- | :--- | :--- | :--- | :--- |
| `aspectRatio` | `"enum"` | Defined by `SOUL_ASPECT` | `"1:1"` | Specifies the output image aspect ratio options. |
| `resolution` | `"enum"` | `["720p", "1080p"]` | `"720p"` | Controls the resolution tier for image generation. |
| `batchSize` | `"enum"` | `["1", "4"]` | `"1"` | Determines the number of images generated per request batch. |
| `enhancePrompt` | `"boolean"` | `true` or `false` | `false` | Toggles automated prompt enhancement logic. |

---

## Exported Model Catalog Entries

The module exports two `ModelEntry` instances. Both entries share the same surface type (`"image"`), roles object (`{}`), and settings configuration (`soulSettings`).

### 1. `soulCinema`

Represents the "Soul Cinema" image model entry.

```typescript
export const soulCinema: ModelEntry
```

* **`id`**: `"soul-cinema"`
* **`surface`**: `"image"`
* **`label`**: `"Soul Cinema"`
* **`roles`**: `{}`
* **`settings`**: References `soulSettings`

### 2. `soul2`

Represents the "Soul 2" image model entry.

```typescript
export const soul2: ModelEntry
```

* **`id`**: `"soul-2"`
* **`surface`**: `"image"`
* **`label`**: `"Soul 2"`
* **`roles`**: `{}`
* **`settings`**: References `soulSettings`

---

## Summary of Usage

This module acts as a declarative catalog registration file. Modules that consume model entries import `soulCinema` or `soul2` to retrieve their identifiers, UI display labels, surface types, and validation schemas for settings parameters.