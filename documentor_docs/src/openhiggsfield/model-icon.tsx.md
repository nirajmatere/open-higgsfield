# Technical Documentation: `src/openhiggsfield/model-icon.tsx`

## Overview

The `src/openhiggsfield/model-icon.tsx` module provides utility functions and a React component designed to map AI model identifier strings to brand icon SVG assets located in `/public/model-icons/`. 

It allows components to resolve the icon file name, construct public image URL paths, and render accessible visual icon placeholders (`<span>` elements) configured via CSS custom properties.

---

## Key Components

### 1. `modelIconFile(id: string): string | undefined`

A pure function that determines the base filename of an icon based on a given model `id` string.

#### Parameters
* **`id`** (`string`): The model identifier string to match.

#### Return Value
* **`string`**: The matching icon filename (without file extension) if a prefix/exact match condition is met.
* **`undefined`**: If the provided `id` does not match any recognized rule.

#### Supported Mapping Logic

| Match Condition | Resolved Icon File Name |
| :--- | :--- |
| `id.startsWith("kling")` | `"kling"` |
| `id.startsWith("wan")` | `"wan"` |
| `id.startsWith("flux")` | `"flux"` |
| `id.startsWith("grok")` | `"grok"` |
| `id.startsWith("happy-horse")` | `"happy-horse"` |
| `id.startsWith("minimax")` | `"minimax"` |
| `id.startsWith("recraft")` | `"recraft"` |
| `id.startsWith("soul")` **OR** `id === "dop"` | `"higgsfield"` |
| `id.startsWith("ideogram")` | `"ideogram"` |
| `id.startsWith("qwen")` | `"qwen"` |
| `id.startsWith("pixverse")` | `"pixverse"` |
| `id.startsWith("ltx")` | `"ltx"` |
| `id.startsWith("z-image")` | `"z-image"` |

---

### 2. `modelIconSrc(id: string): string | undefined`

A utility function that builds the relative URL path to the model icon SVG asset.

#### Parameters
* **`id`** (`string`): The model identifier string.

#### Return Value
* **`string`**: The path string formatted as `/model-icons/${file}.svg` if a valid icon file is found via `modelIconFile(id)`.
* **`undefined`**: If `modelIconFile(id)` returns `undefined`.

---

### 3. `ModelIcon` Component

A React component that renders an inline `<span>` element configured to display the corresponding model icon via CSS custom properties.

#### Props Interface

| Prop | Type | Default Value | Description |
| :--- | :--- | :--- | :--- |
| `modelId` | `string` | *(Required)* | The model identifier used to resolve the icon source path. |
| `size` | `number` | `16` | Sets the `width` and `height` style attributes in pixels. |
| `className` | `string` | `"ohf-model-icon"` | CSS class name applied to the root `<span>` element. |

#### Component Behavior & Rendering Details

1. **Resolution**: Calls `modelIconSrc(modelId)` to retrieve the icon URL path.
2. **Conditional Rendering**:
   * If `src` resolves to `undefined`, the component returns `null` and renders nothing.
3. **DOM Structure**: If resolved, it renders a `<span>` element with the following attributes:
   * **`className`**: Applied class string (defaults to `"ohf-model-icon"`).
   * **`aria-hidden`**: Set to hide the icon element from screen readers.
   * **`style`**: An inline style object containing:
     * `width`: Numerical value in pixels (`size`).
     * `height`: Numerical value in pixels (`size`).
     * `--ohf-model-icon`: Custom CSS variable set to `url("${src}")`.

---

## Code Example Summary

```tsx
import { ModelIcon, modelIconSrc, modelIconFile } from "src/openhiggsfield/model-icon";

// Function usage
const iconName = modelIconFile("kling-v1"); // "kling"
const iconSrc = modelIconSrc("flux-realism"); // "/model-icons/flux.svg"

// React Component usage
function ModelBadge() {
  return (
    <div>
      {/* Renders a 16x16 span with class "ohf-model-icon" and --ohf-model-icon: url("/model-icons/kling.svg") */}
      <ModelIcon modelId="kling-v1" />

      {/* Renders a 24x24 span with custom class */}
      <ModelIcon modelId="wan-video" size={24} className="custom-icon-class" />
    </div>
  );
}
```