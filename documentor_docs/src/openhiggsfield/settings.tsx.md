# Documentation Guide: `src/openhiggsfield/settings.tsx`

## Overview

The `src/openhiggsfield/settings.tsx` module provides Client-side React components for rendering interactive controls for generation model settings. It includes controls for toggling boolean flags, selecting values from enums, and adjusting numeric ranges. 

The module exports two main components:
* **`SettingPill`**: Renders a compact trigger button on the control bar. If the setting is a boolean, clicking the pill directly toggles the setting value. For other setting types (e.g., enums or numeric ranges), clicking the pill triggers a popover menu.
* **`SettingPopover`**: Renders the detailed selection overlay (dialog popover) containing either an option list for enum settings or a slider for numeric range settings.

---

## Dependencies & Imports

* **Next.js Directive**: `"use client";` marks this module as a Next.js Client Component.
* **Types**:
  * `ModelEntry` (imported from `@/generation/catalog`): Represents model schema metadata, including its settings definition (`model.settings`).
* **State Management**:
  * `useSettings` (imported from `@/generation/stores/settings`): Hook used to write setting updates via `settings.set(modelId, { [settingKey]: value })`.
* **Helpers**:
  * `ratioBox`, `settingLabel`, `settingPillLabel`, `settingValueLabel` (imported from `./data`): Formatting and UI label helpers.
* **Icons**:
  * `AudioIcon`, `ClockIcon`, `FormatIcon`, `GemIcon` (imported from `./icons`): Visual icons rendered alongside specific setting controls.
* **UI Components**:
  * `Field`, `OptionList`, `Slider` (imported from `./ui`): Reusable UI primitives for form controls.

---

## Internal Helper Functions

### `glyphFor(key: string, value: unknown)`

Maps a setting key and optional setting value to its corresponding icon or aspect ratio graphic. Returns a React node or `null`.

* **`aspectRatio`**: Renders a custom aspect ratio preview container (`span.ohf-ctl-ratio`). It calls `ratioBox(value)` to calculate visual dimensions and applies CSS class `ohf-ctl-ratio-box` (and conditionally `ohf-ctl-ratio-box--auto` if no explicit ratio box style is returned).
* **`resolution`**: Returns `<GemIcon size={13} />`.
* **`duration`**: Returns `<ClockIcon size={13} />`.
* **`outputFormat`**: Returns `<FormatIcon size={13} />`.
* **`generateAudio`**, **`sound`**, **`keepOriginalSound`**: Returns `<AudioIcon size={13} />`.
* **Default**: Returns `null` if the key does not match any predefined setting icon mapping.

---

## Component Documentation

### 1. `SettingPill`

A UI element representing a model setting on the composer's control bar.

#### Props

| Prop Name | Type | Description |
| :--- | :--- | :--- |
| `model` | `ModelEntry` | The selected model object containing catalog configuration (`model.settings`). |
| `settingKey` | `string` | The identifier for the setting field in `model.settings`. |
| `values` | `Record<string, unknown>` | Current key-value pairs of active user settings. |
| `open` | `boolean` | Indicates whether the popover dialog associated with this pill is currently open. |
| `onOpen` | `(trigger: HTMLElement) => void` | Callback triggered when a non-boolean pill is clicked, passing the element target. |

#### Behavior & Render Logic

1. Retrieves the field configuration via `model.settings[settingKey]`. If the field does not exist in `model.settings`, it renders `null`.
2. Resolves the field label using `settingLabel(settingKey)` and the visual icon via `glyphFor(settingKey, value)`.
3. **Boolean Setting Type (`field.type === "boolean"`)**:
   * Evaluates state: `const on = value === true`.
   * Renders a `<button>` with class `ohf-ctl ohf-ctl--flag`.
   * Set accessibility attributes: `aria-pressed={on}`, `aria-label={`${label} — ${on ? "on" : "off"}`}`.
   * `onClick`: Toggles the boolean state by calling `settings.set(model.id, { [settingKey]: !on })`.
   * Label displays `settingPillLabel(settingKey)`.
4. **Other Setting Types (Non-boolean)**:
   * Renders a `<button>` with class `ohf-ctl ohf-tip` and dataset attribute `data-tip={label}`.
   * Set accessibility attributes: `aria-expanded={open}`, `aria-haspopup="dialog"`, `aria-label={`${label} — ${settingValueLabel(settingKey, value)}`}`.
   * `onClick`: Invokes `onOpen(event.currentTarget)` to open the popover dialog.
   * Label displays `settingValueLabel(settingKey, value)`.

---

### 2. `SettingPopover`

Renders the selection popover interface corresponding to a specific non-boolean setting.

#### Props

| Prop Name | Type | Description |
| :--- | :--- | :--- |
| `model` | `ModelEntry` | The model catalog entry containing settings configuration. |
| `settingKey` | `string` | The key of the active setting to render within the popover. |
| `values` | `Record<string, unknown>` | Current active setting values map. |

#### Behavior & Render Logic

1. Retrieves the field configuration via `model.settings[settingKey]`. 
2. Returns `null` if the field does not exist or if `field.type === "boolean"`.
3. Resolves the human-readable setting label via `settingLabel(settingKey)`.
4. **Enum Setting Type (`field.type === "enum"`)**:
   * Determines the current string value from `values[settingKey]` or defaults to `field.default`.
   * Renders a `<div role="dialog">` with CSS classes `ohf-popover ohf-popover--setting ohf-popover--list`.
   * Renders a `<Field label={label}>` containing an `<OptionList>` component.
   * `<OptionList>` options are generated by mapping `field.values` to objects with `value` and `label` (formatted via `settingValueLabel`).
   * Pass `ratio={settingKey === "aspectRatio"}` to `OptionList`.
   * Handles selection changes via `onChange`, updating settings using `settings.set(model.id, { [settingKey]: next })`.
5. **Numeric / Range Setting Type (Default Non-Enum Branch)**:
   * Determines the current numeric value from `values[settingKey]` or defaults to `field.default`.
   * Renders a `<div role="dialog">` with CSS classes `ohf-popover ohf-popover--setting ohf-scroll`.
   * Renders a `<Field>` with `label` and formatted `value`.
   * Contains a `<Slider>` component initialized with:
     * `min={field.min}`
     * `max={field.max}`
     * `step={field.step ?? 1}`
     * `value={value}`
     * `label={label}`
   * Handles slider adjustment via `onChange`, updating settings using `settings.set(model.id, { [settingKey]: next })`.

---

## Data Flow Summary

```
[ Model Settings Catalog ]
          │
          ├─────────────────────────┐
          ▼                         ▼
   SettingPill               SettingPopover
          │                         │
          ├───────── Direct ────────┤
          │         Updates         │
          ▼                         ▼
   [ user interaction ] ──► [ settings.set(model.id, ...) ] ──► [ useSettings Store ]
```