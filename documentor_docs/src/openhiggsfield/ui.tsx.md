# Technical Documentation: `src/openhiggsfield/ui.tsx`

## Overview

The `src/openhiggsfield/ui.tsx` module is a React client component library containing reusable UI components for form fields, option selection lists, and range sliders. It provides visual layout primitives and interactive controls tailored for settings and parameter selection interfaces.

---

## Directives & Dependencies

- **Client Directive**: Declared as `"use client"` to enable React client-side rendering hooks (`useLayoutEffect`, `useRef`).
- **React Imports**: `useLayoutEffect`, `useRef`, `CSSProperties`, `ReactNode`.
- **Internal Dependencies**:
  - `ratioBox`: Imported from `./data`. Used to resolve aspect ratio box style properties for given aspect ratio values.
  - `CheckIcon`: Imported from `./icons`. SVG icon component rendered to indicate active option selections.

---

## Components Summary

| Component | Export Type | Description |
| :--- | :--- | :--- |
| `Field` | `export` | Wrapper component that renders a labeled field container with optional value header and nested children. |
| `Option` | Internal | Renders an individual button option inside a list, supporting optional aspect ratio preview boxes and selected state indicators. |
| `OptionList` | `export` | Displays a scrollable vertical list of `Option` items, auto-centering the currently selected item on layout render. |
| `Slider` | `export` | Wraps a standard HTML range input (`<input type="range">`), dynamically injecting a `--fill` CSS variable representing percentage progress. |

---

## Detailed Component Specifications

### 1. `Field`

A structural container component designed to present a label, an optional value display, and child components.

#### Props

```typescript
{
  label: string;
  value?: string;
  children: ReactNode;
}
```

#### Behavior & Render Logic

- Renders an outer `div` element with the class `ohf-field`.
- If `value` is `undefined`:
  - Renders a single label element: `<div className="ohf-field-label">{label}</div>`.
- If `value` is provided (string):
  - Renders a row container `<div className="ohf-field-row">` containing:
    - The label: `<div className="ohf-field-label">{label}</div>`
    - The value: `<div className="ohf-field-value">{value}</div>`
- Renders `children` beneath the label/row header.

---

### 2. `Option` *(Internal Component)*

A internal button component representing a single selectable choice within an `OptionList`.

#### Props

```typescript
{
  value: string;
  label: string;
  active: boolean;
  onSelect: () => void;
  ratio?: boolean;
}
```

#### Render Logic

- Renders a `<button>` with:
  - `type="button"`
  - `className="ohf-opt"`
  - `aria-pressed={active}`
  - `onClick={onSelect}`
- **Aspect Ratio Box Representation (`ratio === true`)**:
  - Calls `ratioBox(value)` from `./data`.
  - Renders a `span` wrapper with `className="ohf-opt-ratio"` and `aria-hidden`.
  - Renders an inner preview `span` with dynamic class:
    - `"ohf-opt-box"` if `ratioBox(value)` returns a truthy value.
    - `"ohf-opt-box ohf-opt-box--auto"` if `ratioBox(value)` returns `null` or `undefined`.
  - Applies `box ?? undefined` as the inline `style`.
- **Label**:
  - Renders `<span className="ohf-opt-label">{label}</span>`.
- **Active State Indicator**:
  - If `active` is `true`, renders `<span className="ohf-opt-check" aria-hidden><CheckIcon size={12} /></span>`.

---

### 3. `OptionList`

Renders a vertical group of selectable options and automatically centers the active option upon layout paint.

#### Props

```typescript
{
  options: readonly { value: string; label: string }[];
  value: string;
  onChange: (next: string) => void;
  ratio?: boolean;
}
```

#### Key Logic & Hooks

- **Container Ref**: `listRef` attached to the wrapper `div`.
- **Auto-Scroll via `useLayoutEffect`**:
  - Finds the active child element using the query selector `[aria-pressed="true"]`.
  - Calculates the top scroll offset to vertically center the active option within the scroll container:
    $$\text{scrollTop} = \max\left(0, \text{offsetTop} - \frac{\text{clientHeight} - \text{offsetHeight}}{2}\right)$$
  - Executes synchronously before browser repaints (`useLayoutEffect`).

#### DOM Structure

```tsx
<div className="ohf-opts ohf-scroll" role="group" ref={listRef}>
  {options.map((option) => (
    <Option
      key={option.value}
      value={option.value}
      label={option.label}
      active={option.value === value}
      ratio={ratio}
      onSelect={() => onChange(option.value)}
    />
  ))}
</div>
```

---

### 4. `Slider`

A custom numeric range input wrapper that computes and injects dynamic CSS custom properties for styling input track fills.

#### Props

```typescript
{
  min: number;
  max: number;
  step?: number; // Defaults to 1
  value: number;
  onChange: (next: number) => void;
  label: string;
}
```

#### Operations & Render Logic

- **Fill Calculation**:
  Calculates dynamic percentage string used as a custom CSS variable `--fill`:
  $$\text{fill} = \left(\left( \frac{\text{value} - \text{min}}{(\text{max} - \text{min}) \text{ or } 1} \right) \times 100\right)\text{.toFixed(1)} + \text{"\%"}$$
- **Render Output**:
  Renders an `<input type="range">` element with:
  - `className="ohf-slider"`
  - `min={min}`
  - `max={max}`
  - `step={step}` (default `1`)
  - `value={value}`
  - `aria-label={label}`
  - `style={{ "--fill": fill } as CSSProperties}`
  - `onChange`: Converts `event.target.value` to a number using `Number(...)` and passes it to the `onChange` callback.