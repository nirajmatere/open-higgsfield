# Technical Documentation: `src/openhiggsfield/asset-picker.tsx`

## Overview

The `src/openhiggsfield/asset-picker.tsx` module implements a client-side React component (`AssetPicker`) and its supporting sub-component (`AssetTile`). It acts as a modal popover/dialog interface that allows users to pick, manage, and assign media assets (images, videos, audio) to input roles required by a generation model.

The asset picker bridges two sources of media:
1. **Uploads**: Files uploaded directly from the user's browser/device.
2. **Generations**: Previously completed generation runs from the application history.

---

## Type Definitions & Interfaces

### Exported Types & Interfaces

None exported directly besides the `AssetPicker` functional component.

### Internal Types & Interfaces

#### `Source`
```typescript
type Source = "uploads" | "generations";
```
Defines the active tab or media shelf being displayed.

#### `Asset`
```typescript
interface Asset {
  url: string;
  kind: AssetKind;
  title: string;
  art?: string;
}
```
Represents an individual asset tile within the grid.
- `url`: Public media URL.
- `kind`: The asset type (`AssetKind`), e.g., image, video, audio.
- `title`: Tooltip title and `alt` text (filename or generation prompt).
- `art`: Optional CSS background gradient styling string used as placeholder art.

---

## Component API: `AssetPicker`

The `AssetPicker` component is the main export of this module.

```typescript
export function AssetPicker(props: {
  model: ModelEntry;
  items: MediaItem[];
  uploads: UploadRecord[];
  history: RunRecord[];
  staged: string | null;
  uploading: boolean;
  onUpload: (role: MediaRole) => void;
  onApply: (role: MediaRole, urls: string[]) => void;
  onClose: () => void;
})
```

### Props

| Prop | Type | Description |
| :--- | :--- | :--- |
| `model` | `ModelEntry` | The selected model definition containing allowed input roles and role capacity limits. |
| `items` | `MediaItem[]` | Currently attached media items already present on the canvas/plane. |
| `uploads` | `UploadRecord[]` | Array of browser-uploaded file records. |
| `history` | `RunRecord[]` | Array of history records from previous generation runs. |
| `staged` | `string \| null` | URL of a newly staged asset uploaded while the picker is open. |
| `uploading` | `boolean` | Flag indicating if a file upload operation is currently in progress. |
| `onUpload` | `(role: MediaRole) => void` | Callback triggered when the user requests a local file upload for a role. |
| `onApply` | `(role: MediaRole, urls: string[]) => void` | Callback invoked to apply selected asset URLs to the current role. |
| `onClose` | `() => void` | Callback invoked to dismiss the asset picker dialog. |

---

## Component Logic and Workflow

### State Management

- **`role`** (`MediaRole`): Tracks the active role being edited (defaults to `defaultRole(model)`).
- **`source`** (`Source | null`): Explicit shelf tab (`"uploads"` or `"generations"`). Defaults to `null` on open or role switch, dynamically resolving based on asset availability.
- **`selected`** (`string[]`): An array of asset URLs currently selected in the picker for the active role. Preserves selection order.
- **`tabsRef`** (`useRef<HTMLDivElement>`): React ref pointing to the tab list element for accessibility and focus control.
- **`seen`** (`useRef<string | null>`): Tracks the `staged` prop value to prevent re-selecting previously staged uploads upon reopening.

### Computed Properties & Memos

- **`roles`**: Available roles derived from the model via `rolesOf(model)`.
- **`kind`**: The expected `AssetKind` for the currently active role (`ROLE_KINDS[role]`).
- **`max`**: Maximum allowed items for the active role (`model.roles[role] ?? 0`).
- **`current`**: Array of URLs already assigned to the current role in the canvas/plane (`urlsOf(items, role)`).
- **`room`**: Remaining capacity for the active role (`Math.max(0, max - selected.length)`).
- **`uploadAssets`**: Filters `uploads` matching `kind`.
- **`runAssets`**: Filters completed `history` records matching `kind` and flattens their URLs into `Asset` objects.
- **`shelf`**: Resolves active tab to `"generations"` if `uploads` is empty and completed `history` assets exist; otherwise defaults to `"uploads"`.
- **`assets`**: Array of assets to display based on `shelf`.
- **`canUpload`**: `true` if `room > 0` and `uploading` is `false`.
- **`applyLabel`**: Dynamically calculated button text depending on added or removed count ("Add...", "Remove...", "Replace...", or "Done").

---

## User Interactions & Functions

### Role Switching (`pickRole`)
```typescript
function pickRole(next: MediaRole)
```
Switches the active media role, resets the shelf source selection, and re-initializes the `selected` state with items attached to that role.

### Automatic Slot Advancement (`advanceOrClose`)
```typescript
function advanceOrClose(filled: MediaRole)
```
Checks if the filled role is `"start"` and whether an `"end"` slot exists and has remaining capacity. If unfilled capacity exists in `"end"`, it auto-advances the picker to the `"end"` role. Otherwise, it invokes `onClose()`.

### Asset Selection (`toggle`)
```typescript
function toggle(url: string)
```
- **Single-slot roles (`max === 1`)**: Toggling an asset immediately replaces the selection, invokes `onApply`, and either advances to the next slot or closes the dialog.
- **Multi-slot roles (`max > 1`)**: Toggles the presence of `url` in the `selected` array without immediately triggering `onApply` or closing the picker.

### Committing Changes (`commit`)
```typescript
function commit(urls: string[])
```
Calls `onApply(role, urls)`. If the maximum capacity for the current role is reached, calls `advanceOrClose(role)`; otherwise, triggers `onClose()`.

### External Staging Effect (`useEffect`)
When `staged` changes to a non-null URL different from `seen.current`:
- If `max === 1`, sets selection to `[staged]` and immediately calls `commit([staged])`.
- If `max > 1`, appends `staged` to `selected` if not already present.

### Accessibility & Keyboard Navigation (`onTabKeyDown`)
Listens for `ArrowLeft` and `ArrowRight` key events on the tab list (`role="tablist"`), navigating between `"uploads"` and `"generations"` tabs and shifting focus to the corresponding DOM element.

---

## Sub-Component: `AssetTile`

A memoized functional component (`memo`) representing an individual media tile in the asset grid.

```typescript
const AssetTile = memo(function AssetTile({
  asset,
  picked,
  blocked,
  onToggle,
}: {
  asset: Asset;
  picked: boolean;
  blocked: boolean;
  onToggle: (url: string) => void;
}))
```

### Tile Rendering Details
- **Art Placeholder**: Renders a `<span className="ohf-asset-art">` with dynamic gradient inline styles (`asset.art`) if present.
- **Media Element**:
  - **Video**: Renders a `<video>` element (`muted`, `loop`, `playsInline`, `preload="metadata"`).
  - **Audio**: Displays an `AudioIcon`.
  - **Image/Other**: Renders a standard lazy-loaded `<img>` tag.
- **Video Hover Behavior**: Hovering (`onMouseEnter`) plays the video preview using a internal `videoRef`. Unhovering (`onMouseLeave`) pauses the video and resets `videoRef.current.currentTime` to `0`.
- **Badges**:
  - Displays `PlayBadgeIcon` on video assets.
  - Displays `CheckIcon` inside `.ohf-asset-mark` when selected (`picked`).
- **Disabled State**: Disabled if `blocked` is `true` (i.e., not selected and no remaining slot capacity in the role).

---

## Helper Functions

### `urlsOf`
```typescript
function urlsOf(items: MediaItem[], role: MediaRole): string[]
```
Filters `items` matching the specified `role` and extracts their `url` properties into an array.

### `emptyCopy`
```typescript
function emptyCopy(source: Source, kind: AssetKind): { title: string; hint: string }
```
Generates UI title and hint copy for empty shelf states based on whether the active shelf is `"uploads"` or `"generations"`, and whether the media kind is `"audio"` or otherwise.

---

## CSS Class Structure Reference

The component renders markup structured around the following CSS class hierarchy:

```text
ohf-popover ohf-popover--assets [role="dialog"]
├── ohf-assets-head
│   ├── ohf-assets-tabs [role="tablist"]
│   │   └── ohf-assets-tab [role="tab"]
│   │       └── ohf-assets-tab-count
│   └── ohf-picker-close (.ohf-icon-btn .ohf-icon-btn--ghost)
├── ohf-assets-roles [role="group"] (Rendered if roles.length > 1)
│   └── ohf-chip ohf-chip--role
│       └── ohf-chip-count
├── ohf-assets-body ohf-scroll
│   ├── ohf-picker-empty (When assets array is empty)
│   │   ├── ohf-picker-empty-ic
│   │   ├── ohf-picker-empty-title
│   │   ├── ohf-picker-empty-hint
│   │   └── ohf-btn-solid (Upload button, uploads shelf only)
│   └── ohf-assets-grid (When assets exist)
│       ├── ohf-asset ohf-asset--upload (Upload button tile, uploads shelf only)
│       └── ohf-asset (AssetTile)
│           ├── ohf-asset-art
│           ├── ohf-asset-media (<video> or <img>)
│           ├── ohf-asset-glyph (AudioIcon container)
│           ├── ohf-asset-kind (PlayBadgeIcon container)
│           └── ohf-asset-mark (CheckIcon container)
└── ohf-assets-foot
    ├── ohf-assets-tally
    └── ohf-btn-accent (Apply/Action button)
```