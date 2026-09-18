# Technical Documentation: `src/app/layout.tsx`

## Overview

The `src/app/layout.tsx` file defines the root layout component (`RootLayout`) for the Next.js application, along with global `metadata` and `viewport` configurations. It establishes the base HTML structure (`<html>` and `<body>`), imports core styles (`./base.css`), and sets site-wide SEO, social media, crawler, and theme parameters.

---

## Key Components & Configuration

### 1. Module Imports

* **`type { Metadata, Viewport }` from `"next"`**: TypeScript type definitions from Next.js for strongly typed metadata and viewport objects.
* **`type { ReactNode }` from `"react"`**: Type definition for React children components.
* **Imports from `"@/site"`**:
  * `SITE_DESCRIPTION`: String constant for the application's default meta description.
  * `SITE_NAME`: String constant representing the name of the application.
  * `SITE_TITLE`: String constant for the default document title.
  * `SITE_URL`: String constant representing the base URL of the site.
  * `STUDIO_BG`: Constant defining the background color hex/value for the studio interface.
  * `openGraphFor`: Utility function generating OpenGraph metadata objects.
  * `twitterFor`: Utility function generating Twitter card metadata objects.
* **`"./base.css"`**: Local stylesheet imported at the root level to apply base global CSS styles.

---

### 2. Metadata Export (`metadata`)

An exported object of type `Metadata` that configures HTML `<head>` tags across the application:

| Property | Value / Structure | Description |
| :--- | :--- | :--- |
| `metadataBase` | `new URL(SITE_URL)` | Sets the base URL object for resolving relative metadata URLs. |
| `title` | `{ default: SITE_TITLE, template: "%s — " + SITE_NAME }` | Sets the default page title and defines a template for sub-pages. |
| `description` | `SITE_DESCRIPTION` | Defines the site's meta description tag. |
| `applicationName` | `SITE_NAME` | Specifies the name of the web application. |
| `creator` | `SITE_NAME` | Sets the creator metadata field. |
| `publisher` | `SITE_NAME` | Sets the publisher metadata field. |
| `referrer` | `"origin-when-cross-origin"` | Configures the HTTP referrer header policy. |
| `formatDetection` | `{ telephone: false, address: false, email: false }` | Explicitly disables auto-detection of phone numbers, addresses, and emails to prevent iOS Safari from automatically turning text patterns (such as prompt text or model IDs) into clickable links. |
| `appleWebApp` | `{ title: SITE_NAME }` | Configures Apple standalone web application behavior and title. |
| `openGraph` | `openGraphFor({ path: "/" })` | Populates OpenGraph metadata using the utility function for the root path (`/`). |
| `twitter` | `twitterFor()` | Populates Twitter card metadata using the helper utility. |
| `robots` | Indexing rules object | Configures search engine crawlers (`index: true`, `follow: true`) and sets `googleBot` specifics (`max-image-preview: "large"`, `max-snippet: -1`, `max-video-preview: -1`). |

---

### 3. Viewport Export (`viewport`)

An exported object of type `Viewport` that configures browser UI and viewport behaviors:

* **`colorScheme`**: Set to `"dark"`.
* **`themeColor`**: Set to `STUDIO_BG`.
* **Design Intent**: Since the studio presents a single uniform dark appearance, browser UI elements (chrome) are pinned to match the studio ground color (`STUDIO_BG`) rather than matching user device light/dark preferences.

---

### 4. Component: `RootLayout`

The default export function that wraps all page content rendered within the application.

#### Props
* **`children`**: `ReactNode` — The active page content or sub-layouts rendered inside the root structure.

#### Rendered HTML Structure
```tsx
<html lang="en">
  <body>{children}</body>
</html>
```

* **`html` element**: Defines the root HTML tag with the language attribute set to English (`lang="en"`).
* **`body` element**: Encloses and renders the `children` passed to the component.