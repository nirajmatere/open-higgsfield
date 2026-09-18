# Technical Documentation: `src/openhiggsfield/artwork.ts`

## Overview

The `src/openhiggsfield/artwork.ts` module generates deterministic, layered CSS gradient artwork and provides a film-grain texture asset. It is used to generate UI elements such as model swatches and placeholder background compositions for media generation states (e.g., loading or failed runs). 

By leveraging a custom string hashing algorithm and a pseudo-random number generator (PRNG), compositions remain consistent across renders given the same seed string, eliminating the need to persist visual coordinates in state or storage.

---

## Key Components & Exported API

### `artFor(surface: Surface, hue: number, seedKey: string): string`

Generates a CSS `background-image` value consisting of multiple layered gradients using the `oklch()` color space.

*   **Parameters:**
    *   `surface` (`Surface`): The generation target surface type (imported from `@/generation/catalog`). If `surface === "video"`, an extra subtle scanline gradient is added as the top layer.
    *   `hue` (`number`): The primary base hue angle (0–360) used to build the color palette.
    *   `seedKey` (`string`): A string seed used to deterministically generate random offsets for positions, angles, and secondary hues.
*   **Returns:** `string` — A comma-separated list of CSS radial and linear gradient functions.

#### Layer Composition Logic
The function builds a stack of gradients:
1.  **Video Overlay (Conditional):** Present only when `surface === "video"`. Adds a `linear-gradient(180deg)` standard horizontal highlight scanline.
2.  **Top Radial Highlight:** `radial-gradient` localized around randomly calculated `x1` and `y1` percentages.
3.  **Secondary Accent Radial:** `radial-gradient` positioned at `x2` and `y2` percentages using computed secondary hue `h2`.
4.  **Core Accent Radial:** Small `radial-gradient` centered around `xc` and `yc` percentages using computed contrast hue `hc`.
5.  **Bottom Shadow Radial:** Large `radial-gradient` at `50% 125%` using deep dark tone derived from hue `h3`.
6.  **Base Linear Gradient:** Background `linear-gradient` defined by a variable `angle` (ranging between 138°–182°) transitioning from base hue `hue` to dark hue `h3`.

---

### `swatchFor(surface: Surface, seed: string): string`

A convenience wrapper around `artFor`. It automatically computes the base `hue` by deriving a hash from the provided `seed` string modulo 360.

*   **Parameters:**
    *   `surface` (`Surface`): The target surface type.
    *   `seed` (`string`): The seed string used both for determining base hue (`hash(seed) % 360`) and seeding PRNG variations.
*   **Returns:** `string` — The result of `artFor(surface, hash(seed) % 360, seed)`.

---

### `GRAIN_URI`

`export const GRAIN_URI: string`

A CSS `url(...)` string containing an inline SVG Data URI for an atmospheric film-grain overlay. 

*   **SVG Details:**
    *   **Dimensions:** 128x128 pixels.
    *   **Filter (`id='n'`):** Uses `<feTurbulence>` (`type="fractalNoise"`, `baseFrequency="0.9"`, `numOctaves="2"`, `stitchTiles="stitch"`) and desaturates it to grayscale using `<feColorMatrix type="saturate" values="0"/>`.
    *   **Element:** A full-width/height `<rect>` rendered with the noise filter applied at `0.5` opacity.

---

## Internal Utility Functions

### `hash(str: string): number`

A deterministic string-hashing algorithm based on the 32-bit FNV-1a hash algorithm.

*   **Logic:**
    *   Initializes hash value `h` to `2166136261` (FNV offset basis).
    *   Iterates through each character in `str`, performing an XOR (`^=`) with the character code and multiplying by `16777619` (FNV prime) using `Math.imul`.
    *   Performs a unsigned right-shift (`>>> 0`) to return an unsigned 32-bit integer.
*   **Signature:** `(str: string) => number`

---

### `rng(seed: number): () => number`

A seedable pseudo-random number generator (PRNG) implementation returning numbers in the range $[0, 1)$.

*   **Logic:**
    *   Accepts a numeric seed (defaults to `1` if `0` or falsy).
    *   Returns a closure function that modifies `s` using bitwise transformations (`Math.imul`, XOR `^`, and shifts) on each execution.
    *   Normalizes the unsigned 32-bit integer result by dividing by $4294967296$ ($2^{32}$).
*   **Signature:** `(seed: number) => () => number`

---

## Dependencies

*   `Surface` imported from `@/generation/catalog`.