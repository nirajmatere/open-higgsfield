# Technical Documentation: `src/app/manifest.ts`

## Overview

The `src/app/manifest.ts` file is a dynamic Web App Manifest generator using Next.js App Router metadata conventions (`MetadataRoute.Manifest`). It exports a default function `manifest()` that defines the configuration used by browsers to install the web application on a user's device (PWA behavior).

---

## Imports

| Import | Source | Description |
| :--- | :--- | :--- |
| `MetadataRoute` | `"next"` | Provides TypeScript types for Next.js metadata routes, specifically `MetadataRoute.Manifest`. |
| `SITE_DESCRIPTION` | `"@/site"` | Constant providing the application's description text. |
| `SITE_NAME` | `"@/site"` | Constant providing the short name of the application. |
| `SITE_TITLE` | `"@/site"` | Constant providing the full title/name of the application. |
| `STUDIO_BG` | `"@/site"` | Constant defining the background color hex code/value for the studio interface. |

---

## Function Export

### `manifest()`

- **Export Type**: Default
- **Return Type**: `MetadataRoute.Manifest`

Generates and returns the manifest configuration object required by Next.js to produce a `manifest.json` file.

```typescript
export default function manifest(): MetadataRoute.Manifest
```

---

## Manifest Configuration Properties

The returned object contains the following attributes:

### Application Metadata
- **`name`** (`SITE_TITLE`): The full name of the web application used in app stores or install prompts.
- **`short_name`** (`SITE_NAME`): The short version of the application name used on device home screens where space is limited.
- **`description`** (`SITE_DESCRIPTION`): General description of the application.
- **`categories`**: An array of application categories describing the site's functionality:
  - `"productivity"`
  - `"graphics"`
  - `"photo"`

### Routing and Display
- **`start_url`**: `"/"` — Specifies the starting URL launched when the user opens the installed application.
- **`scope`**: `"/"` — Defines the navigation scope of the installed application.
- **`display`**: `"standalone"` — Configures the application to open in standalone mode, running without browser UI controls.

### Visual Styling
- **`background_color`**: `STUDIO_BG` — Sets the background color of the app before stylesheets are loaded.
- **`theme_color`**: `STUDIO_BG` — Sets the OS UI toolbar/theme color.

> **Implementation Note**: Both `background_color` and `theme_color` are explicitly set to `STUDIO_BG` because when installed, the application opens directly into the studio interface rather than the marketing page chrome.

### Icons Configuration

The `icons` array defines four icon assets available for different screen resolutions and display contexts:

| Asset Source | Sizes | MIME Type | Purpose |
| :--- | :--- | :--- | :--- |
| `/icon.svg` | `"any"` | `image/svg+xml` | `any` |
| `/icon-192.png` | `"192x192"` | `image/png` | `any` |
| `/icon-512.png` | `"512x512"` | `image/png` | `any` |
| `/icon-maskable-512.png` | `"512x512"` | `image/png` | `maskable` |