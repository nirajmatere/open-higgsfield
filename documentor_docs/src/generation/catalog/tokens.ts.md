# Technical Documentation: `src/generation/catalog/tokens.ts`

## Overview

The `src/generation/catalog/tokens.ts` file acts as a static token repository containing predefined, immutable array constants for aspect ratios. These constants define valid set values for two aspect ratio sets: `SOUL_ASPECT` and `SEEDANCE_ASPECT`.

---

## Constants & Key Components

### 1. `SOUL_ASPECT`

An immutable (`as const`) array defining supported aspect ratio strings for the `SOUL` category or configuration.

* **Export Name:** `SOUL_ASPECT`
* **Type:** `readonly ["9:16", "16:9", "4:3", "3:4", "1:1", "2:3", "3:2"]`
* **Values:**
  * `"9:16"`
  * `"16:9"`
  * `"4:3"`
  * `"3:4"`
  * `"1:1"`
  * `"2:3"`
  * `"3:2"`

```typescript
export const SOUL_ASPECT = ["9:16", "16:9", "4:3", "3:4", "1:1", "2:3", "3:2"] as const;
```

---

### 2. `SEEDANCE_ASPECT`

An immutable (`as const`) array defining supported aspect ratio strings for the `SEEDANCE` category or configuration.

* **Export Name:** `SEEDANCE_ASPECT`
* **Type:** `readonly ["16:9", "4:3", "1:1", "3:4", "9:16", "21:9"]`
* **Values:**
  * `"16:9"`
  * `"4:3"`
  * `"1:1"`
  * `"3:4"`
  * `"9:16"`
  * `"21:9"`

```typescript
export const SEEDANCE_ASPECT = ["16:9", "4:3", "1:1", "3:4", "9:16", "21:9"] as const;
```

---

## Technical Details & Behavior

* **Const Assertion (`as const`):** 
  Both arrays use TypeScript's `as const` assertion. This guarantees:
  1. The arrays are read-only (`readonly`) at runtime and compile-time; elements cannot be added, removed, or mutated.
  2. The string elements are inferred as narrow literal types (e.g., `"16:9"`) rather than general `string` types.
  3. The array retains its exact position and length as a tuple.