# OpenHiggsfield Icon Set Documentation (`src/openhiggsfield/icons.tsx`)

## Overview

The `src/openhiggsfield/icons.tsx` module provides a comprehensive collection of SVG icon components for the OpenHiggsfield application design system. All icons are rendered as React components designed around a 16×16 coordinate grid (`viewBox="0 0 16 16"`), utilizing `currentColor` for consistent styling and theming.

---

## Design System Specifications

* **Grid Base**: 16×16 coordinate system (`viewBox="0 0 16 16"`).
* **Default Stroke Width**: `1.5px` (with explicit exceptions set to `1.8px`).
* **Line Caps & Joins**: Round (`strokeLinecap="round"`, `strokeLinejoin="round"`).
* **Color Handling**: Inherits typography color using `currentColor`.
* **Accessibility**: All icons include `aria-hidden="true"` by default.
* **Fills**: Mainly standard strokes (`fill="none"`), with filled elements reserved for active states or media badges (e.g., `PlayIcon`, `PlayBadgeIcon`, and `HeartIcon` when `filled={true}`).

---

## Type Definitions & Helper Functions

### `IconProps`

Interface for standard icon props.

```typescript
interface IconProps {
  size?: number;
}
```

* **`size`** (`number`, optional): Sets both the `width` and `height` of the rendered SVG in pixels. Each icon component provides a default value if not specified.

---

### `base(size: number)`

A private helper function that generates standard SVG attributes for stroked line icons.

```typescript
function base(size: number)
```

**Returned Object Configuration:**
* `width`: `size`
* `height`: `size`
* `viewBox`: `"0 0 16 16"`
* `fill`: `"none"`
* `stroke`: `"currentColor"`
* `strokeWidth`: `1.5`
* `strokeLinecap`: `"round"`
* `strokeLinejoin`: `"round"`
* `aria-hidden`: `true`

---

### Private Constants

#### `HEART`
A string defining the SVG path geometry for the heart icon shape:
`"M8 13.2C8 13.2 2.4 9.75 2.4 6.05C2.4 4.28 3.78 3.05 5.4 3.05C6.5 3.05 7.5 3.65 8 4.55C8.5 3.65 9.5 3.05 10.6 3.05C12.22 3.05 13.6 4.28 13.6 6.05C13.6 9.75 8 13.2 8 13.2Z"`

---

## Exported Icon Components

Below is the complete reference of all 30 icon components exported by this file.

### 1. `ImageIcon`
* **Default Size**: `16`
* **Description**: Image placeholder icon containing a frame, sun element, and mountain outline.

### 2. `VideoIcon`
* **Default Size**: `16`
* **Description**: Video/media player frame containing a solid filled play triangle.

### 3. `AudioIcon`
* **Default Size**: `16`
* **Description**: Audio waveform consisting of four vertical bars.

### 4. `AssetsIcon`
* **Default Size**: `16`
* **Description**: A stack of asset frames seen edge-on, representing stored runs or collections.

### 5. `KeyIcon`
* **Default Size**: `15`
* **Description**: Key symbol featuring a circular head and notched blade.

### 6. `CaretDownIcon`
* **Default Size**: `10`
* **Description**: Downward-pointing chevron indicator.

### 7. `CloseIcon`
* **Default Size**: `14`
* **Description**: "X" mark composed of two diagonal intersecting paths.

### 8. `CheckIcon`
* **Default Size**: `13`
* **Special Properties**: Overrides base `strokeWidth` to `1.8`.
* **Description**: Checkmark symbol.

### 9. `SearchIcon`
* **Default Size**: `14`
* **Description**: Magnifying glass search symbol.

### 10. `ShuffleIcon`
* **Default Size**: `14`
* **Description**: Circular arrow loop with arrow head indicating shuffle/reload.

### 11. `RetryIcon`
* **Default Size**: `13`
* **Description**: Counter-clockwise circular arrow indicating retry.

### 12. `WarningIcon`
* **Default Size**: `14`
* **Description**: Alert triangle with an exclamation mark inside.

### 13. `ArrowRightIcon`
* **Default Size**: `13`
* **Description**: Right-pointing horizontal arrow.

### 14. `PlusIcon`
* **Default Size**: `14`
* **Description**: Add/plus sign cross.

### 15. `MinusIcon`
* **Default Size**: `14`
* **Description**: Remove/minus horizontal bar.

### 16. `GemIcon`
* **Default Size**: `14`
* **Description**: Diamond/gem shape representing quality or resolution grades, complete with a girdle line.

### 17. `ClockIcon`
* **Default Size**: `14`
* **Description**: Circle clock face with hour and minute hands.

### 18. `FormatIcon`
* **Default Size**: `14`
* **Description**: Document icon with a folded top-right corner.

### 19. `PlayIcon`
* **Default Size**: `18`
* **Special Implementation**: Does not use `base()`. Directly renders an SVG with a solid filled path (`fill="currentColor"`).

### 20. `PlayBadgeIcon`
* **Default Size**: `9`
* **Special Implementation**: Compact version of the play triangle. Does not use `base()`. Renders a solid filled path (`fill="currentColor"`).

### 21. `DownloadIcon`
* **Default Size**: `13`
* **Description**: Downward arrow entering a base platform line.

### 22. `OpenOutIcon`
* **Default Size**: `13`
* **Description**: External link icon showing an arrow exiting an open corner frame. Used when redirecting to a new tab.

### 23. `UploadIcon`
* **Default Size**: `13`
* **Description**: Upward arrow departing a base platform line (mirrors `DownloadIcon`).

### 24. `CopyIcon`
* **Default Size**: `13`
* **Description**: Two overlapping/offset sheets indicating a duplicate or copy action.

### 25. `SlidersIcon`
* **Default Size**: `14`
* **Description**: Horizontal adjustment sliders with circular handles.

### 26. `ArrowUpIcon`
* **Default Size**: `15`
* **Special Properties**: Overrides base `strokeWidth` to `1.8`.
* **Description**: Upward-pointing vertical arrow.

### 27. `HeartIcon`
* **Default Size**: `14`
* **Additional Props**: `filled?: boolean` (defaults to `false`).
* **Description**: Heart symbol for saving or liking runs. When `filled` is true, sets `fill="currentColor"`; otherwise `fill="none"`. Uses the static `HEART` path.

### 28. `TrashIcon`
* **Default Size**: `13`
* **Description**: Trash can icon with lid, top handle, and tapered body container.

### 29. `UndoIcon`
* **Default Size**: `13`
* **Description**: Clockwise circular arc with arrow pointing left/back.

### 30. `WaveBadgeIcon`
* **Default Size**: `12`
* **Description**: Compact wave indicator composed of four vertical lines.

---

## Component Summary Matrix

| Component Name | Default Size | Uses `base()` | Custom Attributes / Notes |
| :--- | :--- | :--- | :--- |
| `ImageIcon` | 16 | Yes | Standard base properties |
| `VideoIcon` | 16 | Yes | Standard base properties |
| `AudioIcon` | 16 | Yes | Standard base properties |
| `AssetsIcon` | 16 | Yes | Standard base properties |
| `KeyIcon` | 15 | Yes | Standard base properties |
| `CaretDownIcon` | 10 | Yes | Standard base properties |
| `CloseIcon` | 14 | Yes | Standard base properties |
| `CheckIcon` | 13 | Yes | `strokeWidth={1.8}` |
| `SearchIcon` | 14 | Yes | Standard base properties |
| `ShuffleIcon` | 14 | Yes | Standard base properties |
| `RetryIcon` | 13 | Yes | Standard base properties |
| `WarningIcon` | 14 | Yes | Standard base properties |
| `ArrowRightIcon` | 13 | Yes | Standard base properties |
| `PlusIcon` | 14 | Yes | Standard base properties |
| `MinusIcon` | 14 | Yes | Standard base properties |
| `GemIcon` | 14 | Yes | Standard base properties |
| `ClockIcon` | 14 | Yes | Standard base properties |
| `FormatIcon` | 14 | Yes | Standard base properties |
| `PlayIcon` | 18 | No | Direct SVG render, `fill="currentColor"` |
| `PlayBadgeIcon` | 9 | No | Direct SVG render, `fill="currentColor"` |
| `DownloadIcon` | 13 | Yes | Standard base properties |
| `OpenOutIcon` | 13 | Yes | Standard base properties |
| `UploadIcon` | 13 | Yes | Standard base properties |
| `CopyIcon` | 13 | Yes | Standard base properties |
| `SlidersIcon` | 14 | Yes | Standard base properties |
| `ArrowUpIcon` | 15 | Yes | `strokeWidth={1.8}` |
| `HeartIcon` | 14 | Yes | Dynamic prop `filled?: boolean`, sets `fill` attribute |
| `TrashIcon` | 13 | Yes | Standard base properties |
| `UndoIcon` | 13 | Yes | Standard base properties |
| `WaveBadgeIcon` | 12 | Yes | Standard base properties |