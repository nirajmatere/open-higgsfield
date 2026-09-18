# Technical Documentation: `src/generation/catalog/types.ts`

## Overview

The `src/generation/catalog/types.ts` file defines core TypeScript types and interfaces used for managing generation models, platform payload path mappings, configurable settings, media role assignments, and runtime generation request structures. 

It serves as the data contract for cataloging models (e.g., image vs. video models) and defining the schema for generation requests.

---

## Type Definitions

### 1. `Surface`
Defines the output target or medium type for a generation model.
```typescript
export type Surface = "image" | "video";
```
* **Possible Values**:
  * `"image"`: Model produces static image outputs.
  * `"video"`: Model produces video outputs.

---

### 2. `MediaRole`
Specifies the functional role that an input media item plays within a generation process.
```typescript
export type MediaRole = "start" | "end" | "reference" | "video" | "audio";
```
* **Possible Values**:
  * `"start"`: Starting frame or media asset.
  * `"end"`: Ending frame or media asset.
  * `"reference"`: Reference asset used for style, character, or context conditioning.
  * `"video"`: Source video input.
  * `"audio"`: Source audio input.

---

### 3. `MediaItem`
Represents an individual media asset supplied to a generation payload.
```typescript
export type MediaItem = {
  id: string;
  url: string;
  role: MediaRole;
};
```
* **Fields**:
  * `id` (`string`): Unique identifier for the media item.
  * `url` (`string`): Web address or URI pointing to the media resource.
  * `role` (`MediaRole`): The designated role of the media item.

---

### 4. `SettingField`
A discriminated union type describing configurable settings/parameters accepted by a model.
```typescript
export type SettingField =
  | { type: "enum"; values: readonly string[]; default: string }
  | { type: "range"; min: number; max: number; default: number; step?: number }
  | { type: "boolean"; default: boolean };
```
* **Variants**:
  * **Enum Setting** (`type: "enum"`):
    * `values`: Array of string options allowed for this setting.
    * `default`: The default selected string value.
  * **Range Setting** (`type: "range"`):
    * `min`: Minimum numeric value.
    * `max`: Maximum numeric value.
    * `default`: Default numeric value.
    * `step` *(optional)*: Incremental step size for the numeric range.
  * **Boolean Setting** (`type: "boolean"`):
    * `default`: Default boolean state (`true` or `false`).

---

### 5. `PlatformPaths`
Maps standardized generation inputs to specific property path strings required by platform-specific API payloads when utilizing a shared mapper.
```typescript
export type PlatformPaths = {
  text?: string;
  image?: string;
  firstLast?: string;
  reference?: string;
};
```
* **Fields** *(all optional)*:
  * `text`: Path key for text prompts.
  * `image`: Path key for single image inputs.
  * `firstLast`: Path key for start/end frame inputs.
  * `reference`: Path key for reference inputs.

---

### 6. `ModelEntry`
Defines the complete catalog entry and schema for a generation model.
```typescript
export type ModelEntry = {
  id: string;
  surface: Surface;
  label: string;
  roles: Partial<Record<MediaRole, number>>;
  settings: Record<string, SettingField>;
  paths?: PlatformPaths;
};
```
* **Fields**:
  * `id` (`string`): Unique model identifier.
  * `surface` (`Surface`): Output type (`"image"` or `"video"`).
  * `label` (`string`): Human-readable display name for the model.
  * `roles` (`Partial<Record<MediaRole, number>>`): Maps supported `MediaRole` keys to numeric limits or counts allowed by the model.
  * `settings` (`Record<string, SettingField>`): Key-value map defining configurable parameters available for the model.
  * `paths` (`PlatformPaths`, *optional*): Submission paths used when the shared mapper is applied (note: models requiring custom maps like Soul, Kling 3, or Seedance omit or override standard paths).

---

### 7. `GenerationPlane`
Represents the runtime data structure containing all user inputs and configurations needed to execute a generation request.
```typescript
export type GenerationPlane = {
  model: string;
  prompt: { text: string };
  media: Partial<Record<MediaRole, MediaItem[]>>;
  settings: Record<string, unknown>;
};
```
* **Fields**:
  * `model` (`string`): The ID of the target model being invoked.
  * `prompt` (`{ text: string }`): Prompt object containing the input text query.
  * `media` (`Partial<Record<MediaRole, MediaItem[]>>`): Input media assets categorized by their assigned `MediaRole`.
  * `settings` (`Record<string, unknown>`): Active runtime key-value settings populated by the user/system for generation execution.

---

## Summary of Relationships

* **`ModelEntry`** relies on **`Surface`** for output classification, **`MediaRole`** to define supported input media limits, **`SettingField`** to define configurable fields, and **`PlatformPaths`** for payload mapping instructions.
* **`GenerationPlane`** holds the concrete runtime inputs matching a `ModelEntry` configuration, linking selected **`MediaItem`** arrays under their respective **`MediaRole`** keys alongside dynamic setting values.