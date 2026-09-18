# Technical Documentation: `src/openhiggsfield/data.ts`

## Overview

The `src/openhiggsfield/data.ts` module provides UI constants, localized labels, formatting helpers, static datasets, and utility functions for managing generation surface states, model specifications, media roles, aspect ratios, and settings presentation within the OpenHiggsfield interface.

---

## Imported Types

The module depends on the following types imported from `@/generation/catalog`:
* `MediaRole`
* `ModelEntry`
* `Surface`

---

## Types and Data Structures

### Types

#### `GalleryView`
```typescript
export type GalleryView = Surface | "assets" | "favorites";
```
Defines the scope for gallery content display:
* `Surface` (`"image"` or `"video"`)
* `"assets"`: All generation runs across both surfaces.
* `"favorites"`: User-favorited generation runs across both surfaces.

#### `AssetKind`
```typescript
export type AssetKind = "image" | "video" | "audio";
```
Represents coarse media asset categories used by file pickers and asset managers.

#### `CountSetting`
```typescript
export type CountSetting = {
  key: string;
  kind: "enum" | "range";
  counts: number[];
};
```
Describes a batch count configuration extracted from a model entry:
* `key`: The parameter name (`"numImages"` or `"batchSize"`).
* `kind`: Indicates whether the count parameter is represented as a `"range"` or `"enum"`.
* `counts`: An ascending list of permissible batch count values.

---

### Constants

#### Surfaces & Views
* `SURFACES: readonly Surface[] = ["image", "video"]` — Supported generation surface identifiers.
* `SURFACE_LABELS: Record<Surface, string>` — Display titles for surfaces (`"Image"`, `"Video"`).
* `VIEWS: readonly GalleryView[] = ["image", "video", "assets", "favorites"]` — Complete set of gallery views.
* `VIEW_LABELS: Record<GalleryView, string>` — Readable labels for gallery views.
* `CROSS_VIEWS: Set<GalleryView>` — Set containing `["assets", "favorites"]`, indicating views that cross both media surfaces.

#### UI Prompts & Samples
* `PROMPT_PLACEHOLDERS: Record<Surface, string>` — Placeholder instructional strings displayed in prompt input textareas per surface type.
* `SAMPLES: Record<Surface, string[]>` — Collections of full prompt strings used to populate empty prompt inputs for `"image"` and `"video"` surfaces.

#### Settings & UI Displays
* `COUNT_KEYS: string[] = ["numImages", "batchSize"]` — Model settings keys that dictate result counts per request.

#### Media Role Configuration
* `ROLE_LABELS: Record<MediaRole, string>` — Full display labels for media roles (`"Start frame"`, `"End frame"`, `"Reference"`, `"Video"`, `"Audio"`).
* `ROLE_TAGS: Record<MediaRole, string>` — Short uppercase abbreviations for attachment tiles (`"START"`, `"END"`, `"REF"`, `"VIDEO"`, `"AUDIO"`).
* `ROLE_KINDS: Record<MediaRole, AssetKind>` — Maps roles to generic asset kinds (`"image"`, `"video"`, or `"audio"`).
* `ROLE_ACCEPT: Record<MediaRole, string>` — Standard MIME type filter strings corresponding to media inputs for file pickers.

---

## Utility Functions

### Prompt Helpers

#### `pickSamples(surface: Surface, count = 3): string[]`
Shuffles the prompt pool for the specified `surface` using an in-place Fisher-Yates algorithm and returns a slice containing `count` random prompt strings. 

*Note: Designed for client-side invocation to prevent server/client hydration mismatches.*

---

### Label & Formatting Helpers

#### `settingLabel(key: string): string`
Returns a human-readable label for a setting key. Looks up the key in `SETTING_LABELS`. If unmapped, converts camelCase key strings to title-cased words (e.g., `aspectRatio` -> `Aspect Ratio`).

#### `settingPillLabel(key: string): string`
Returns concise UI pill labels from `SETTING_PILL_LABELS` (e.g., mapping `generateAudio` to `"Audio"`). Falls back to `settingLabel(key)` if no compact label is explicitly defined.

#### `settingValueLabel(key: string, value: unknown): string`
Converts raw setting values into UI-ready string representations:
* `boolean`: Returns `"On"` or `"Off"`.
* `number`: Appends `"s"` if `key === "duration"`, otherwise returns stringified number.
* `"auto"`: Capitalized to `"Auto"`.
* Format patterns matching `/^\d+k$/` or `key === "outputFormat"`: Transformed to uppercase (e.g., `"4k"` -> `"4K"`, `"png"` -> `"PNG"`).

#### `roleNoun(role: MediaRole, count: number): string`
Generates plural or singular nouns for media attachment interface elements. Returns lowercased `ROLE_LABELS` when `count === 1`, or the corresponding plural form (defined in `ROLE_PLURALS`) when `count !== 1`.

#### `formatClock(totalSeconds: number): string`
Formats a numeric duration in seconds into `M:SS` time format (e.g., `125` -> `"2:05"`).

#### `durationBadge(values: Record<string, unknown>): string | undefined`
Extracts `values.duration` if it is a `number` and returns its `formatClock` string representation; otherwise returns `undefined`.

#### `metaOf(model: ModelEntry, values: Record<string, unknown>): string`
Constructs a summary text chip line by extracting generation metadata values:
* For `image` surface models: Includes `resolution` and `outputFormat`.
* For `video` surface models: Includes `resolution`, `duration`, and `"Audio"` (if `generateAudio` is `true`).
* Formats defined properties with `settingValueLabel` and joins non-empty items with `" · "`.

---

### Media Role Utilities

#### `rolesOf(model: ModelEntry): MediaRole[]`
Extracts defined media roles keys from a `ModelEntry` instance.

#### `defaultRole(model: ModelEntry): MediaRole`
Determines the primary/fallback role for a model:
1. Returns `"reference"` if the surface is `"image"` and `model.roles.reference` exists.
2. Returns `"start"` if `model.roles.start` exists.
3. Otherwise, returns the first available role from `rolesOf(model)`, defaulting to `"reference"`.

---

### Aspect Ratio Helpers

#### `ratioBox(value: string): { width: number; height: number } | null`
Parses aspect ratio formatted strings (`"W:H"`). Scales dimensions down into a bounding box of maximum dimension `14px`, enforcing a minimum width/height constraint of `5px`. Returns `null` if `value` does not match the `W:H` regex format.

#### `ratioToCss(value: unknown, fallback: string): string`
Parses an aspect ratio string matching `"W:H"` and formats it as standard CSS syntax (`"W / H"`). Returns `fallback` if parsing fails.

---

### Model Descriptor Helpers

#### `describeModel(model: ModelEntry): string`
Generates a plain-text description summarizing a model's inputs and limits:
1. Identifies base output type (`"Images"` vs `"Video"`).
2. Deduplicates input requirement phrases based on roles (`ROLE_PHRASES`).
3. Summarizes maximum resolution ceilings (takes the highest item from `resolution.values`) and duration ranges (`min`–`max`s).
4. Combines requirements and limits into a formatted string (e.g., `"Images from a prompt, references · to 4K"`).

#### `countSetting(model: ModelEntry): CountSetting | null`
Scans a model's settings against `COUNT_KEYS` (`"numImages"`, `"batchSize"`). Returns a `CountSetting` object containing valid output count options if configured as a numeric `"range"` or numeric `"enum"`. Returns `null` if no count keys are defined on the model settings.