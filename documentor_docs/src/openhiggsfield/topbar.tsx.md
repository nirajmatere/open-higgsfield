# Technical Documentation: `src/openhiggsfield/topbar.tsx`

The `topbar.tsx` file exports the `Topbar` client-side React component. It serves as the top navigation bar for the OpenHiggsfield AI UI, providing gallery scope tab navigation and a control button for platform API key configuration.

---

## Overview

The `Topbar` component renders:
1. An accessible heading for screen readers (`h1.ohf-sr`).
2. A tabbed navigation bar (`role="tablist"`) allowing users to switch gallery views (`image`, `video`, `assets`, `favorites`) with full keyboard accessibility and an animated, dynamic measurement indicator ("thumb").
3. A platform key status button that visually indicates key configuration and busy state, allowing users to trigger a modal to add or edit their key.

---

## Component Interface

### Props

The `Topbar` component accepts a single inline props object:

| Prop | Type | Description |
| :--- | :--- | :--- |
| `view` | `GalleryView` | The currently selected gallery view scope (`image`, `video`, `assets`, or `favorites`). |
| `onView` | `(next: GalleryView) => void` | Callback function invoked when the user selects or navigates to a new view. |
| `busy` | `boolean` | Boolean indicating whether a background operation or generation is actively processing. Applied to the key button via `data-busy`. |
| `keyConfigured` | `boolean` | Boolean indicating whether a platform API key is configured. Controls key button label, accessibility attributes, and `data-ready`. |
| `onKeys` | `() => void` | Callback function triggered when clicking the platform key button. |

---

## Internal State & Refs

### State

- **`thumb`** (`{ x: number; w: number } | null`): Holds the `offsetLeft` (`x`) and `offsetWidth` (`w`) of the currently active tab element. Defaults to `null` until measured.

### Refs

- **`tabsRef`** (`useRef<HTMLDivElement>(null)`): Holds a reference to the `div.ohf-tabs` container element to perform DOM queries, measure dimensions, and manage focused elements during keyboard navigation.

---

## Internal Utilities & Mappings

### `VIEW_ICONS`

A static record mapping each `GalleryView` type to a render function for its icon:

```tsx
const VIEW_ICONS: Record<GalleryView, () => React.ReactNode> = {
  image: () => <ImageIcon />,
  video: () => <VideoIcon />,
  assets: () => <AssetsIcon />,
  favorites: () => <HeartIcon size={15} />,
};
```

---

## Key Functionalities

### 1. Dynamic Active Tab Indicator Measurement

Instead of assuming fixed-width columns, the active tab indicator (`span.ohf-thumb`) dynamically measures the active tab element (`[aria-selected="true"]`) to exact pixel dimensions.

- **Measurement Mechanism**:
  - `useEffect` triggers whenever `view` changes.
  - Measures `active.offsetLeft` and `active.offsetWidth` to update the `thumb` state.
  - Uses `ResizeObserver` attached to `tabsRef.current` to adjust measurements if the container resizes.
  - Subscribes to `document.fonts.ready` to ensure tab widths are correctly remeasured after custom web fonts finish loading and cause potential layout reflows.
- **CSS Custom Properties**:
  - The measurement is passed directly to the indicator `span` as inline style variables: `--thumb-x` and `--thumb-w`.
  - Sets `data-ready={thumb !== null}` to signal rendering readiness.

### 2. ARIA Keyboard Navigation (`onKeyDown`)

The component implements WAI-ARIA tablist keyboard navigation rules within `onKeyDown`:

- **Supported Keys**:
  - **`ArrowRight`**: Focuses and selects the next tab (wraps around to the first tab if at the end).
  - **`ArrowLeft`**: Focuses and selects the previous tab (wraps around to the last tab if at the start).
  - **`Home`**: Focuses and selects the first tab (index `0`).
  - **`End`**: Focuses and selects the last tab (`VIEWS.length - 1`).
- **Behavior**:
  - Prevents default scroll behavior when navigating mapped keys.
  - Calls `onView(VIEWS[to])` with the target view.
  - Focuses the underlying HTML `<button>` element using standard DOM `.focus()`.

### 3. Gallery Tab Selection

Renders a list of tab buttons by iterating over `VIEWS`:
- Each tab button has `role="tab"` and controls the panel `ohf-panel`.
- `aria-selected` is set based on `view === id`.
- Unselected tabs receive `tabIndex={-1}`, while the active tab receives `tabIndex={0}` (roving tabindex pattern).
- `aria-label` is set to `VIEW_LABELS[id]`.
- For the `favorites` view specifically, a `title` attribute is also applied (`title={VIEW_LABELS[favorites]}`).

### 4. Platform Key Button

Displays the status of the user's platform key:
- **Button Data Attributes**:
  - `data-busy`: Reflects the `busy` prop boolean.
  - `data-ready`: Reflects the `keyConfigured` prop boolean.
- **Dynamic Content & Accessibility**:
  - Text label: Displays `"Your key"` when `keyConfigured` is `true`, otherwise `"Add key"`.
  - `aria-label` / `title`: Displays `"Edit platform key"` when `keyConfigured` is `true`, otherwise `"Add platform key"`.
- **Elements**:
  - Rendered with `<KeyIcon />`, `<span className="ohf-key-text">`, and a indicator bulb element `<span className="ohf-lamp" />`.
  - Fires the `onKeys` prop callback on click.

---

## HTML & Component Structure

```tsx
<div className="ohf-topbar">
  <h1 className="ohf-sr">OpenHiggsfield AI — Open source AI studio</h1>

  {/* Tab Navigation Section */}
  <div className="ohf-bar ohf-enter-1">
    <div className="ohf-tabs" role="tablist" aria-label="Gallery scope" ref={tabsRef} onKeyDown={onKeyDown}>
      <span className="ohf-thumb" data-ready={...} aria-hidden style={{ "--thumb-x": "...", "--thumb-w": "..." }} />
      {/* Map through VIEWS */}
      <button role="tab" className="ohf-tab" ...>
        {VIEW_ICONS[id]()}
        <span className="ohf-tab-label">{VIEW_LABELS[id]}</span>
      </button>
    </div>
  </div>

  {/* Key Status Section */}
  <div className="ohf-bar ohf-enter-1">
    <button className="ohf-key" data-busy={busy} data-ready={keyConfigured} onClick={onKeys} ...>
      <KeyIcon />
      <span className="ohf-key-text">{keyConfigured ? "Your key" : "Add key"}</span>
      <span className="ohf-lamp" />
    </button>
  </div>
</div>
```