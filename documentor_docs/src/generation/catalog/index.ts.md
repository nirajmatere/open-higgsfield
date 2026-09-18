# Technical Documentation: `src/generation/catalog/index.ts`

## Overview

The `src/generation/catalog/index.ts` module acts as the central registry and export index for generation models and associated types/utilities within the codebase. It aggregates individual model configurations from separate submodules into a unified catalog, provides a utility function to query models by their unique identifier, and re-exports key type definitions and helper functions.

---

## Key Components

### 1. `MODELS` Array

```typescript
export const MODELS: readonly ModelEntry[] = [ ... ];
```

- **Type:** `readonly ModelEntry[]`
- **Description:** A read-only array containing all supported generation model entries registered in the catalog. 
- **Registered Model Instances:**
  - `soul2`, `soulCinema` (from `./soul`)
  - `seedance25`, `seedance25Edit`, `seedance25Extend` (from `./seedance-2.5`)
  - `seedance2`, `seedance2Fast`, `seedance2Mini` (from `./seedance-2`)
  - `kling3Turbo`, `kling3Std`, `kling3Pro`, `kling34k`, `kling3MotionStd`, `kling3MotionPro` (from `./kling-3`)
  - `flux2` (from `./flux-2`)
  - `grokImagine2` (from `./grok-imagine-2`)
  - `ideogram4` (from `./ideogram-4`)
  - `recraft41` (from `./recraft-4.1`)
  - `qwenImage3` (from `./qwen-image-3`)
  - `zImageTurbo` (from `./z-image-turbo`)
  - `wan3` (from `./wan-3`)
  - `wan3Prime` (from `./wan-3-prime`)
  - `wan27` (from `./wan-2.7`)
  - `wan26` (from `./wan-2.6`)
  - `flux3` (from `./flux-3`)
  - `minimaxH3` (from `./minimax-h3`)
  - `minimaxHailuo23` (from `./minimax-hailuo-2.3`)
  - `happyHorse1` (from `./happy-horse-1`)
  - `happyHorse11` (from `./happy-horse-1.1`)
  - `kling26` (from `./kling-2.6`)
  - `kling25` (from `./kling-2.5`)
  - `klingO3` (from `./kling-o3`)
  - `klingO1` (from `./kling-o1`)
  - `ltx25Fast` (from `./ltx-2.5-fast`)
  - `ltx25Pro` (from `./ltx-2.5-pro`)
  - `grokImagineVideo15` (from `./grok-imagine-video-1.5`)
  - `pixverse6` (from `./pixverse-6`)
  - `dop` (from `./dop`)

---

### 2. `getModel` Function

```typescript
export function getModel(id: string): ModelEntry
```

- **Description:** Looks up and returns a single `ModelEntry` from the `MODELS` array matching the supplied string `id`.
- **Parameters:**
  - `id` (`string`): The unique identifier of the target model entry.
- **Returns:** `ModelEntry` matching the provided `id`.
- **Error Handling:** Throws an `Error` with the message ``Unknown model: ${id}`` if no matching model is found in `MODELS`.

---

### 3. Re-exported Utility Functions

- **`parseSettings`**: Re-exported from `./parse-settings`.

---

### 4. Re-exported Types

The module re-exports the following TypeScript interfaces and types directly from `./types`:

- `GenerationPlane`
- `MediaItem`
- `MediaRole`
- `ModelEntry`
- `PlatformPaths`
- `Surface`

---

## Import / Export Dependency Summary

| Source Module | Exported / Aggregated Items |
| :--- | :--- |
| `./dop` | `dop` |
| `./flux-2` | `flux2` |
| `./flux-3` | `flux3` |
| `./grok-imagine-2` | `grokImagine2` |
| `./grok-imagine-video-1.5` | `grokImagineVideo15` |
| `./happy-horse-1` | `happyHorse1` |
| `./happy-horse-1.1` | `happyHorse11` |
| `./ideogram-4` | `ideogram4` |
| `./kling-2.5` | `kling25` |
| `./kling-2.6` | `kling26` |
| `./kling-3` | `kling34k`, `kling3MotionPro`, `kling3MotionStd`, `kling3Pro`, `kling3Std`, `kling3Turbo` |
| `./kling-o1` | `klingO1` |
| `./kling-o3` | `klingO3` |
| `./ltx-2.5-fast` | `ltx25Fast` |
| `./ltx-2.5-pro` | `ltx25Pro` |
| `./minimax-h3` | `minimaxH3` |
| `./minimax-hailuo-2.3` | `minimaxHailuo23` |
| `./parse-settings` | `parseSettings` |
| `./pixverse-6` | `pixverse6` |
| `./qwen-image-3` | `qwenImage3` |
| `./recraft-4.1` | `recraft41` |
| `./seedance-2` | `seedance2`, `seedance2Fast`, `seedance2Mini` |
| `./seedance-2.5` | `seedance25`, `seedance25Edit`, `seedance25Extend` |
| `./soul` | `soul2`, `soulCinema` |
| `./types` | `GenerationPlane`, `MediaItem`, `MediaRole`, `ModelEntry`, `PlatformPaths`, `Surface` |
| `./wan-2.6` | `wan26` |
| `./wan-2.7` | `wan27` |
| `./wan-3` | `wan3` |
| `./wan-3-prime` | `wan3Prime` |
| `./z-image-turbo` | `zImageTurbo` |