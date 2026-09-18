# Technical Documentation: `src/openhiggsfield/selection-bar.tsx`

## Overview

The `selection-bar.tsx` module exports a client-side React component (`SelectionBar`) designed to handle bulk actions on selected items (referred to as "runs" or `RunRecord`s). 

It provides action buttons for bulk operations—such as downloading, favoriting, deleting, and clearing the selection—along with a visual indicator stack representing the currently selected items.

---

## File Metadata & Imports

- **Client Component Directive**: Declared with `"use client";`.
- **React Core**: Imports `useRef` and type `CSSProperties`.
- **Local Imports**:
  - `RunRecord` imported from `./history`.
  - Icon components (`CloseIcon`, `DownloadIcon`, `HeartIcon`, `PlayBadgeIcon`, `TrashIcon`) imported from `./icons`.

---

## Constants

### `STACK`
```typescript
const STACK = 3;
```
Defines the maximum number of items (run records) rendered visually in the stack display (`.ohf-selstack`) before truncation.

---

## Types & Interfaces

### `SaveProgress`
Represents the current progress state when saving or downloading items.

```typescript
export interface SaveProgress {
  done: number;  // Number of completed items
  total: number; // Total number of items to save
}
```

### Component Props (`SelectionBar` Props)
The `SelectionBar` component accepts an inline object type with the following properties:

| Property | Type | Description |
| :--- | :--- | :--- |
| `records` | `RunRecord[]` | Array of currently selected run records. |
| `saving` | `SaveProgress \| null` | Download/save progress state, or `null` if idle. |
| `onDownload` | `() => void` | Callback function triggered when clicking the Download button. |
| `onFavorite` | `() => void` | Callback function triggered when clicking the Favorite button. |
| `onDelete` | `() => void` | Callback function triggered when clicking the Delete button. |
| `onClose` | `() => void` | Callback function triggered when clearing the selection. |

---

## Component Logic & State Handling

### Persistent Selection (`useRef`)
To prevent visual glitches (such as displaying "0 selected") during exit transitions/animations when the selection is cleared:

```typescript
const on = records.length > 0;
const held = useRef(records);
if (on) held.current = records;
const shown = held.current;
```

- **`on`**: Boolean flag indicating if there is an active selection (`records.length > 0`).
- **`held`**: A React ref holding the most recent non-empty `records` array.
- **`shown`**: Evaluates to `held.current`, ensuring that the bar retains the selection context while animating away after selection is cleared.

### Derived Values

- **`count`**: Total number of items in `shown`.
- **`saveable`**: Count of items in `shown` that have a valid primary URL (`urls[0]`).
- **`allKept`**: Boolean checking if every record in `shown` has `favorite === true`.
- **`noun`**: Pluralized string (`"run"` if `count === 1`, otherwise `"runs"`).
- **`downloadLabel`**: Formatted string used for tooltips and accessibility labels:
  - When saving: `"Saving {done} of {total}"`
  - When `saveable === 0`: `"Nothing here to download"`
  - When `saveable < count`: `"Download {saveable} of {count}"`
  - Otherwise: `"Download"`

---

## Component Structure & Render Tree

The element tree uses custom class names prefixed with `ohf-` and specific ARIA attributes.

```
div.ohf-selbar [role="group", aria-label="Bulk actions", data-on, inert]
├── p.ohf-selbar-count [role="status"]
│   ├── span.ohf-selstack [aria-hidden]
│   │   └── span.ohf-selchip (repeated up to STACK times)
│   │       ├── img (if kind === "image" && urls[0])
│   │       └── span.ohf-selchip-play > PlayBadgeIcon (if kind === "video")
│   └── span.ohf-selbar-text
│       ├── span.ohf-selbar-n (key={count})
│       └── " selected"
├── span.ohf-selbar-rule [aria-hidden]
├── span.ohf-selbar-slot.ohf-tip [data-tip=downloadLabel]
│   └── button.ohf-selact.ohf-selact--wide [disabled, aria-label, onClick=onDownload]
│       ├── span.ohf-spinner OR DownloadIcon
│       └── span.ohf-selact-label
├── button.ohf-selact.ohf-tip [data-tip, data-on, aria-pressed, aria-label, onClick=onFavorite]
│   └── HeartIcon
├── button.ohf-selact.ohf-selact--danger.ohf-tip [data-tip, aria-label, onClick=onDelete]
│   └── TrashIcon
├── span.ohf-selbar-rule [aria-hidden]
└── button.ohf-selact.ohf-selact--quiet.ohf-tip.ohf-tip--end [data-tip, aria-label, onClick=onClose]
    └── CloseIcon
```

---

## Key Features & Action Buttons

### 1. Item Visualizer & Count (`ohf-selbar-count`)
- Renders up to the last 3 items (`STACK`) using `shown.slice(-STACK)`.
- Each item is styled as a chip (`.ohf-selchip`) setting a CSS custom property `--z` for dynamic z-indexing and an inline `background` color (`record.art`).
- If the item is an image with a valid `urls[0]`, an `<img>` tag is rendered.
- If the item is a video, it renders a `PlayBadgeIcon` within `.ohf-selchip-play`.
- Displays the total count dynamically (`count`) next to the text `"selected"`.

### 2. Bulk Download Button
- **Disabled Conditions**: Disabled if `saveable === 0` or if `saving !== null`.
- **Icon**: Displays a `span.ohf-spinner` when `saving` is active, otherwise displays `DownloadIcon` (size 15).
- **Label Text**: Displays progress as `${saving.done}/${saving.total}` when `saving`, or `"Download"` when idle.

### 3. Bulk Favorite Button
- **State Behavior**: Sets `aria-pressed={allKept}` and `data-on={allKept}`.
- **Icon**: Renders `HeartIcon` (size 16) with `filled={allKept}`.
- **Tooltip / Label**:
  - If `allKept` is `true`: `"Remove {count} {noun} from favorites"` / `"Remove from favorites"`.
  - If `allKept` is `false`: `"Save {count} {noun} to favorites"` / `"Save to favorites"`.

### 4. Bulk Delete Button
- Styled with `.ohf-selact--danger`.
- Triggers `onDelete` on click.
- Features a `TrashIcon` (size 15).
- Tooltip/Label: `"Delete {count} {noun}"`.

### 5. Clear Selection Button
- Styled with `.ohf-selact--quiet`.
- Triggers `onClose` on click.
- Features a `CloseIcon` (size 14).
- Tooltip: `"Clear selection · Esc"`.

---

## Accessibility Details

- **Container Role**: Uses `role="group"` with `aria-label="Bulk actions"` rather than `role="toolbar"` to preserve standard tab-navigation behavior.
- **`inert` Attribute**: Applied as `inert={!on}` to prevent focus or interaction when no items are active/selected.
- **Screen Reader Notifications**: The selection count container (`p.ohf-selbar-count`) uses `role="status"` to announce changes in selection count automatically.
- **Decorative Elements**: Decorative stack spans and horizontal rule dividers specify `aria-hidden`.