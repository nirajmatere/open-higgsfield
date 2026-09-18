# Technical Documentation: `src/site.ts`

## Overview

The `src/site.ts` file serves as the single source of truth for the application's brand identity, canonical origin URL, background color tokens, and default Open Graph and Twitter metadata configurations. 

This module is designed for **server-only usage**. Because certain environment variables (such as `VERCEL_PROJECT_PRODUCTION_URL`) are not exposed to the browser, importing this module into client components (`"use client"`) will cause origin resolution to evaluate differently between the client and server.

---

## Constants

### Identity & Branding

* **`SITE_NAME`** (`string`)
  * **Value:** `"OpenHiggsfield AI"`
  * The primary name of the site.

* **`SITE_DESCRIPTOR`** (`string`)
  * **Value:** `"Open source AI studio"`
  * A brief descriptor tag line used in combination with the site name.

* **`SITE_TITLE`** (`string`)
  * **Value:** `${SITE_NAME} — ${SITE_DESCRIPTOR}` (`"OpenHiggsfield AI — Open source AI studio"`)
  * The full formatted title string.

* **`SITE_DESCRIPTION`** (`string`)
  * **Value:** `"A studio for image and video generation — one prompt bar, each model’s own settings, and every finished run in one gallery."`
  * Default description describing the site's functionality.

* **`STUDIO_BG`** (`string`)
  * **Value:** `"#0a0a0b"`
  * Color hex code representing the near-black background of the studio ground, used also as the installed-app and browser-chrome color.

---

### Canonical Origin & URLs

* **`SITE_URL`** (`string`)
  * Evaluates to the string returned by calling `resolveOrigin()`. Used as the canonical origin for the site.

---

### Social Media & Asset Configuration

* **`OG_IMAGE`** (`object`)
  * Structure holding the default Open Graph image attributes:
    * `url`: `"/og.png"`
    * `width`: `1200`
    * `height`: `630`
    * `type`: `"image/png"`
    * `alt`: `"The OpenHiggsfield AI open-frame mark on a near-black field, above the OpenHiggsfield AI wordmark, the words Open source AI studio, and a line describing one prompt bar for image and video with every finished run in one gallery."`

---

## Functions

### `resolveOrigin()`

An internal helper function that resolves the base origin URL for the site using environment variables with a local fallback.

#### Resolution Hierarchy:
1. **Explicit Custom URL:** Checks `process.env.NEXT_PUBLIC_SITE_URL`. If present (and not empty after trimming), trailing slashes are removed and the string is returned.
2. **Vercel Production URL:** Checks `process.env.VERCEL_PROJECT_PRODUCTION_URL`. If present (and not empty after trimming), any existing `http://` or `https://` protocol prefix and trailing slashes are removed, and it returns the URL formatted as `https://<domain>`.
3. **Fallback URL:** Returns `"http://localhost:3000"` if neither environment variable is defined.

#### Return Value
* `string`: The resolved site origin without a trailing slash.

---

### `openGraphFor(options)`

Generates an Open Graph metadata object for a specific route. This prevents Next.js from dropping default fields (`type`, `siteName`, `locale`, `images`) when a route defines custom Open Graph metadata.

#### Parameters

An object containing:
* `path` (`string`): The URL path for the specific route.
* `title` (`string`, optional): Custom title. Defaults to `SITE_TITLE`.
* `description` (`string`, optional): Custom description. Defaults to `SITE_DESCRIPTION`.

#### Return Value

Returns an object matching the Open Graph metadata structure:

```typescript
{
  type: "website",
  siteName: "OpenHiggsfield AI",
  locale: "en_US",
  url: path,
  title: string,
  description: string,
  images: [ OG_IMAGE ]
}
```

---

### `twitterFor(options)`

Generates a Twitter card metadata object for social media sharing.

#### Parameters

An optional object containing:
* `title` (`string`, optional): Custom title. Defaults to `SITE_TITLE`.
* `description` (`string`, optional): Custom description. Defaults to `SITE_DESCRIPTION`.

#### Return Value

Returns an object formatted for Twitter card metadata:

```typescript
{
  card: "summary_large_image",
  title: string,
  description: string,
  images: [ OG_IMAGE ]
}
```