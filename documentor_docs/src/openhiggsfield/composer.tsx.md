# Technical Documentation: `src/openhiggsfield/composer.tsx`

## Overview

The `src/openhiggsfield/composer.tsx` file defines the **Composer** interface component for the OpenHiggsfield client. It serves as the primary dock and control panel for prompt input, model selection, model configuration settings, batch size adjustment, asset attachments, and generation execution.

It is implemented as a Client Component (`"use client"`) and includes features such as:
- Dynamic prompt textarea auto-expansion based on text length and container width.
- Dynamic popover anchor positioning and overlay management (Model Picker, Asset Picker, Setting Popovers).
- Native vs. Studio batch count resolution.
- Dock height measurement and propagation via CSS custom properties.
- Keyboard navigation and shortcut handling (`⌘↵` / `Ctrl↵`, Escape key overlay dismissal, ARIA spinbutton controls).

---

## Module Constants & Helper Functions

### Constants

* `PICKER` (`"picker"`): Identifier for opening the `ModelPicker` overlay.
* `ASSETS` (`"assets"`): Identifier for opening the `AssetPicker` overlay.
* `SETTING` (`"setting:"`): String prefix for dynamic setting popover identifiers.
* `PROMPT_MAX_HEIGHT` (`168`): Maximum height (in pixels) for the prompt `textarea` before scrolling.
* `STUDIO_COUNTS`: Array from `1` to `MAX_BATCH` representing fallback batch sizes when a model does not declare native count settings.
* `POPOVER_GAP` (`8`): Breathing room (in pixels) between a popover panel and its trigger element.

### Helper Functions

#### `popoverWidth(id: string, model: ModelEntry): number`
Calculates the pixel width reserved for an overlay/popover based on its ID and model settings:
- Returns `560` for `PICKER` and `ASSETS`.
- Returns `216` for settings of type `"enum"`.
- Returns `268` for all other settings (e.g., sliders).

---

## Components

### 1. `Composer`

The top-level component that renders the floating generation dock, prompt input area, setting controls, asset attachments, and generation trigger button.

#### Component Props (`Composer`)

| Prop | Type | Description |
| :--- | :--- | :--- |
| `surface` | `Surface` | The surface mode (e.g., `"image"` or `"video"`). Determines which prompt store (`useImagePrompt` or `useVideoPrompt`) and placeholder text to use. |
| `model` | `ModelEntry` | The active model catalog entry containing settings configuration and label information. |
| `generating` | `boolean` | Flag indicating if a generation process is currently in progress. |
| `error` | `string \| null` | An error message string to display inside an alert banner, or `null`. |
| `focusNonce` | `number` | Incremental trigger value used to programmatically focus the prompt textarea. |
| `history` | `RunRecord[]` | List of historical generation runs, passed down to `AssetPicker`. |
| `notice` | `ReactNode` (optional) | Rendered notice node positioned at the top of the composer wrapper. |
| `selection` | `ReactNode` | Selection interface/toolbar element that shares a swap slot with the composer. |
| `selecting` | `boolean` | State indicating whether bulk selection mode is active. Automatically closes overlays when `true`. |
| `onError` | `(message: string \| null) => void` | Callback to clear or update the error state. |
| `onGenerate` | `() => void` | Callback triggered when user submits a generation request. |

#### Internal State & Hooks
- **Prompt State**: Extracted dynamically based on `surface` using `useImagePrompt()` or `useVideoPrompt()`.
- **Active Model Store**: `useActive()` provides `setModel`, `batch`, and `setBatch`.
- **Settings Store**: `useSettings()` manages model-specific settings state parsed via `parseSettings()`.
- **Media Tray**: `useMediaTray(model, onError)` manages attached assets, upload handlers, and staged items.
- **Overlay State**:
  - `overlay`: `string | null` representing active open popover (`PICKER`, `ASSETS`, or `setting:<key>`).
  - `anchor`: `{ x: number, y: number }` coordinates for popover positioning.
- **OS Shortcut Detection**: `shortcut` state stores `"⌘↵"` for macOS/iOS or `"Ctrl↵"` for other operating systems.

#### Key Side Effects (`useEffect`)

1. **Dock Height Observer**:
   Measures `dockRef.current.offsetHeight` using a `ResizeObserver` and sets `--ohf-dock-h` on the dock's parent element to dynamically adapt layout spacing.

2. **Outside Click & Escape Dismissal**:
   Attaches global `pointerdown` and `keydown` listeners while an overlay is active. Dismisses `overlay` if a click occurs outside `wrapRef` or if `Escape` is pressed.

3. **Prompt Textarea Auto-Grow**:
   Adjusts `promptRef.current.style.height` based on `scrollHeight` (capped at `PROMPT_MAX_HEIGHT`). A `ResizeObserver` recalculates height when container width changes without looping on height updates.

4. **Programmatic Focus**:
   Focuses `promptRef` whenever `focusNonce` changes and is greater than `0`.

5. **Selection Mode Lockout**:
   Resets `overlay` to `null` whenever `selecting` becomes `true`.

6. **Platform Hotkey Detection**:
   Checks `navigator.userAgent` post-mount to assign either `"⌘↵"` or `"Ctrl↵"`.

#### Dynamic Popover Alignment (`toggle`)

The `toggle(next: string, trigger: HTMLElement)` function controls popover visibility and position:
- Toggles off if the clicked control matches the open `overlay`.
- Computes trigger coordinates relative to `wrapRef`.
- Clamps the calculated X offset within container bounds using `popoverWidth()`.
- Accounts for `.ohf-strip` (MediaStrip) presence when computing the ceiling Y boundary so popovers do not obscure input attachments.
- Sets CSS variables `--ohf-pop-x` and `--ohf-pop-y` on `wrapRef`.

---

### 2. `BatchStepper`

A dedicated stepper control component for modifying the batch generation size (results per request).

#### Component Props (`BatchStepper`)

| Prop | Type | Description |
| :--- | :--- | :--- |
| `value` | `number` | Current batch size integer. |
| `counts` | `number[]` | Allowed batch counts specified by the model or `STUDIO_COUNTS`. |
| `onChange` | `(next: number) => void` | Callback invoked with the new batch value. |

#### Keyboard & Accessibility
- Exposes an element with `role="spinbutton"` and ARIA attributes (`aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext`).
- Supports keyboard navigation on the value element:
  - `ArrowUp` / `ArrowRight`: Step increment.
  - `ArrowDown` / `ArrowLeft`: Step decrement.
  - `Home`: Select minimum available count (`counts[0]`).
  - `End`: Select maximum available count (`counts[last]`).

---

## Batch Value Resolution Logic

The `Composer` determines batch functionality by evaluating whether a model defines a native batch/count setting via `countSetting(model)`:

```typescript
const native = countSetting(model);
const counts = native ? native.counts : STUDIO_COUNTS;
const batchValue = native ? Number(values[native.key]) || counts[0]! : batch;
const settingKeys = Object.keys(model.settings).filter((key) => key !== native?.key);
```

- **If Native Count Exists**: The control writes directly to model settings via `settings.set(model.id, ...)`. The setting key is excluded from rendering as a separate `SettingPill`.
- **If No Native Count**: The control updates the global active studio batch store via `setBatch()`.

---

## DOM Structure Overview

```html
<div class="ohf-dock" ref={dockRef} data-selecting={selecting}>
  <div class="ohf-composer-wrap ohf-enter-2" ref={wrapRef} style="--ohf-pop-x: ...; --ohf-pop-y: ...;">
    <!-- Optional Notice -->
    <!-- Optional Alert Error Banner -->
    
    <!-- Active Popover / Overlay -->
    <!-- SettingPopover | AssetPicker | ModelPicker -->

    <div class="ohf-swap">
      <div class="ohf-composer">
        <!-- Media Strip Component -->
        <MediaStrip model={model} />

        <div class="ohf-composer-head">
          <!-- Optional Attachment Button (Plus Icon) -->
          <!-- Prompt Textarea -->
          <textarea class="ohf-prompt" ... />
        </div>

        <div class="ohf-composer-row">
          <div class="ohf-controls">
            <!-- Model Selection Trigger Button -->
            <!-- Model Setting Pills (SettingPill) -->
            <!-- Batch Size Stepper (BatchStepper) -->
          </div>

          <!-- Generate Action Button -->
          <button class="ohf-generate" disabled={disabled} data-busy={generating} ...>
            <span class="ohf-generate-sheen" /> <!-- Rendered when generating -->
            <ArrowUpIcon />
            <span>Generate</span>
            <kbd class="ohf-kbd">⌘↵</kbd>
          </button>
        </div>
      </div>

      <!-- Selection Element (rendered parallel in swap slot) -->
      {selection}
    </div>
  </div>
</div>
```