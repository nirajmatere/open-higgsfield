## Technical Documentation: `src/openhiggsfield/gallery.tsx`

### Overview

The `src/openhiggsfield/gallery.tsx` module implements the main gallery user interface for the OpenHiggsfield client application. It renders a 4-column, virtualized grid displaying active render jobs (`ActiveRun`) and past output records (`RunRecord`). 

The module manages video hover previews, multi-item selection modes, dynamic row height calculations using container size observation, empty gallery states with dynamic sample starters, and elapsed time tracking for ongoing renders.

---

## Core Constants & Types

### Grid Configurations
* **`GRID_GAP`** (`14`): Gap in pixels between grid tiles.
* **`COLUMNS`** (`4`): Fixed number of columns in the gallery grid layout.
* **`CARD_RATIO`** (`4 / 3`): Aspect ratio (4:3) allocated for each grid slot card.

### Types
* **`Slot`**: A discriminated union representing grid slot contents:
  * `{ key: string; kind: "run"; run: ActiveRun }`: Slot containing an in-progress generation task.
  * `{ key: string; kind: "item"; item: RunRecord; index: number }`: Slot containing a completed or failed historical record.

### Messaging Dictionary (`EMPTY`)
Defines titles and hints displayed when no items or active runs exist for a specific `GalleryView` (`image`, `video`, `assets`, `favorites`).

---

## Helper Functions & Custom Hooks

### `shortPrompt(prompt: string): string`
Truncates prompts to a maximum length of 48 characters, appending an ellipsis (`…`) if the prompt exceeds 47 characters. Used to generate accessible ARIA labels and titles.

### `slotsOf(runs: ActiveRun[], items: RunRecord[]): Slot[]`
Transforms and combines active runs and historical items into a unified array of `Slot` items. Any historical item in `items` with `status === "running"` is converted into a `run` slot.

### `runSubtitle(item: RunRecord): string`
Formated relative timestamp for a run item by invoking `timeAgo(item.createdAt)`.

### `useInnerWidth(ref: RefObject<HTMLElement | null>): number`
A React hook that calculates and tracks the inner content width (excluding horizontal padding) of a container element.
* Uses a `ResizeObserver` to re-measure on element resize.
* Returns `0` if the DOM node reference is unavailable.

---

## Components

### 1. `Gallery` (Exported Component)
The top-level container for gallery views. Wrapped in `React.memo`.

#### Props
| Prop | Type | Description |
| :--- | :--- | :--- |
| `view` | `GalleryView` | Active view category (`image`, `video`, `assets`, `favorites`). |
| `surface` | `Surface` | Active generation surface type. |
| `items` | `RunRecord[]` | List of historical generation runs. |
| `runs` | `ActiveRun[]` | List of currently active render tasks. |
| `freshIds` | `string[]` | Array of recently added item IDs (for entrance animations). |
| `picked` | `ReadonlySet<string>` | Set of selected item IDs. |
| `onOpen` | `(id: string) => void` | Callback when a card is clicked to open the detail viewer. |
| `onPick` | `(id: string, index: number, range: boolean) => void` | Selection toggle callback (receives `shiftKey` state for range selection). |
| `onReuse` | `(item: RunRecord) => void` | Callback to populate prompt/settings from a run. |
| `onFavorite` | `(item: RunRecord) => void` | Callback to toggle an item's favorite state. |
| `onDownload` | `(item: RunRecord) => Promise<void>` | Callback to trigger media file download. |
| `onDelete` | `(item: RunRecord) => void` | Callback to remove an item. |
| `onStarter` | `(prompt: string) => void` | Callback when a sample starter prompt is clicked. |
| `galleryRef` | `RefObject<HTMLDivElement \| null>` | Reference attached to the main scrollable element. |

#### Behavior
* Evaluates whether `items` and `runs` are both empty. If empty, renders the `<Empty>` component.
* If items or runs exist, renders `<VirtualizedGrid>`.
* Implements `role="tabpanel"` accessible panel markup.

---

### 2. `VirtualizedGrid`
Renders a virtualized list of rows using TanStack Virtual (`@tanstack/react-virtual`).

#### Mechanics
* Calls `useInnerWidth(scrollRef)` to get dynamic grid width.
* Groups `Slot` array elements into rows of 4 (`COLUMNS`).
* Calculates dynamic row heights via `estimateSize`:
  $$\text{Row Height} = \frac{\text{Column Width}}{\text{CARD\_RATIO}} + \text{GRID\_GAP}$$
* Configured with an `overscan` value of 4 rows.
* Calls `virtualizer.measure()` within an effect when `width` or `slots` change.
* Maps active slots to either `<RunningTile>` or `<Tile>`.

---

### 3. `Tile`
Renders individual media cards (images, videos, or failed status states). Wrapped in `React.memo`.

#### Internal State
* `beat` (`boolean`): Triggered on favorite action click to execute heart beat CSS animations.
* `saving` (`boolean`): Async loading state for download actions.

#### Key Features
* **Failed State Handling**: Renders a specialized `.ohf-tile--failed` container showing model info, error string, action buttons (`Retry this run`, `Delete run`), and selection checkbox.
* **Video Hover Preview**:
  * Attaches `onMouseEnter` to play the video (`videoRef.current.play()`).
  * Attaches `onMouseLeave` to pause and reset time (`currentTime = 0`).
* **Selection Mode Interactivity**:
  * Offers explicit check-box controls (`aria-checked`).
  * If `selecting` is true (when `picked.size > 0`), clicking the main card body triggers selection (`onPick`) instead of opening the viewer (`onOpen`).
* **Metadata Overlay**: Renders prompt text, `ModelIcon`, model label string, and metadata strings parsed from `item.meta` split by `" · "`.
* **Action Rail Controls**:
  * **Delete**: Invokes `onDelete(item)`.
  * **Reuse/Retry**: Invokes `onReuse(item)`.
  * **Download**: Invokes `onDownload(item)`, toggling the loading spinner (`saving`).
  * **Favorite**: Invokes `onFavorite(item)` and triggers heart beat animation.

---

### 4. `RunningTile`
Renders a skeleton card placeholder for active, ongoing render operations (`ActiveRun`).

#### Functionality
* Tracks and displays live elapsed render time in `MM:SS` format.
* Runs a 1-second interval timer updated based on `run.startedAt`.
* Carries `role="status"` and accessible ARIA attributes indicating rendering state.

---

### 5. `Empty`
Displays empty gallery state instructions and prompt starter recommendations.

#### Features
* Displays specific header title and hint text from the `EMPTY` dictionary based on `view`.
* Excludes starter list if `CROSS_VIEWS.has(view)` returns true (i.e., views like `assets` or `favorites`).
* Loads dynamic prompt samples via `pickSamples(surface)` after component mounting (avoids SSR hydration mismatch).
* Renders starter prompts alongside visual swatch gradients generated via `swatchFor(surface, sample)`.
* Triggers `onStarter(sample)` upon clicking any starter option.