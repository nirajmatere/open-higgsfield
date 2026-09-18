# File Documentation: `src/app/page.tsx`

## Overview

The `src/app/page.tsx` file serves as the main entry point and route component for the root path (`/`) in a Next.js App Router application. 

Its primary purpose is to:
1. Configure and load the Google "Inter" font using Next.js font optimization.
2. Define route-specific metadata (specifically setting the canonical URL).
3. Import required CSS styles for the page module.
4. Render the `OpenHiggsfieldApp` component, passing the font's CSS variable class name as a prop.

---

## Code Breakdown & Key Components

### 1. Imports

```typescript
import type { Metadata } from "next";
import { Inter } from "next/font/google";

import { OpenHiggsfieldApp } from "@/openhiggsfield/openhiggsfield-app";

import "@/openhiggsfield/openhiggsfield.css";
```

* **`type { Metadata }`**: Imported from `next`. Used to type-check the route's exported `metadata` object.
* **`Inter`**: Imported from `next/font/google`. A font loader function used to optimize and load the Inter Google Font.
* **`OpenHiggsfieldApp`**: Imported from `@/openhiggsfield/openhiggsfield-app`. This is the core React component rendered by this page route.
* **`@/openhiggsfield/openhiggsfield.css`**: Side-effect import that includes the CSS stylesheet required for the `OpenHiggsfieldApp` surface.

---

### 2. Font Configuration

```typescript
const inter = Inter({
  subsets: ["latin"],
  variable: "--font-ohf-inter",
  display: "swap",
});
```

Initializes the `Inter` font instance with the following settings:
* **`subsets: ["latin"]`**: Pre-loads only the Latin character subset.
* **`variable: "--font-ohf-inter"`**: Defines a CSS custom property (variable) name for the font family, allowing it to be referenced in CSS stylesheets.
* **`display: "swap"`**: Uses the CSS `font-display: swap` strategy to ensure text remains visible using a fallback font while the custom font is loading.

---

### 3. Route Metadata

```typescript
export const metadata: Metadata = {
  alternates: { canonical: "/" },
};
```

* **`metadata`**: An exported `Metadata` object recognized by Next.js to inject head elements for SEO.
* **`alternates.canonical`**: Sets the canonical link rel tag to `"/"` for this specific route.

---

### 4. Page Component (`OpenHiggsfieldPage`)

```typescript
export default function OpenHiggsfieldPage() {
  return <OpenHiggsfieldApp fontClassName={inter.variable} />;
}
```

* **`OpenHiggsfieldPage`**: The default export function representing the page component for the `/` route.
* **Props Passed**:
  * **`fontClassName={inter.variable}`**: Passes the generated CSS variable class name from the `inter` font configuration instance to the `OpenHiggsfieldApp` component.

---

## How It Works

1. **Page Request**: When a user accesses the root path (`/`), Next.js evaluates `src/app/page.tsx`.
2. **Metadata Processing**: Next.js processes the exported `metadata` object and outputs a `<link rel="canonical" href="/" />` tag in the document head.
3. **Styles & Font Injection**: The imported `openhiggsfield.css` stylesheet is included on the page. The `Inter` font config creates a CSS variable (`--font-ohf-inter`) attached to a class name accessible via `inter.variable`.
4. **Rendering**: The `OpenHiggsfieldPage` function executes and returns the `<OpenHiggsfieldApp />` component initialized with the `fontClassName` prop set to `inter.variable`.