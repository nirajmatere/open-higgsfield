# Technical Documentation: `src/app/sitemap.ts`

## Purpose

The `src/app/sitemap.ts` file is a Next.js App Router metadata file used to automatically generate a `sitemap.xml` for search engine optimization (SEO). It defines the sitemap entry for the application's root URL, specifying its update frequency, priority, and last modification timestamp.

---

## Code Overview

```typescript
import type { MetadataRoute } from "next";

import { SITE_URL } from "@/site";

export default function sitemap(): MetadataRoute.Sitemap {
  const lastModified = new Date();

  return [{ url: `${SITE_URL}/`, lastModified, changeFrequency: "weekly", priority: 1 }];
}
```

---

## Dependencies & Imports

*   **`MetadataRoute`** (`from "next"`):
    *   A type definition provided by Next.js.
    *   Used to strictly type the output of the default `sitemap()` function (`MetadataRoute.Sitemap`).
*   **`SITE_URL`** (`from "@/site"`):
    *   A constant representing the base URL of the site.
    *   Used to construct absolute URLs for sitemap entries.

---

## Component Breakdown

### Export Default Function: `sitemap()`

The default export function constructs and returns the sitemap configuration array.

#### Internal Variables

*   **`lastModified`** (`Date`):
    *   Instantiates a new JavaScript `Date` object (`new Date()`) representing the current timestamp when the function executes.

#### Return Value

The function returns an array containing a single sitemap entry object matching the `MetadataRoute.Sitemap` type structure:

| Property | Type | Value | Description |
| :--- | :--- | :--- | :--- |
| `url` | `string` | `${SITE_URL}/` | The absolute URL for the root page of the site. |
| `lastModified` | `Date` | `lastModified` | The date and time when the page was last modified (defaults to the execution time). |
| `changeFrequency` | `string` | `"weekly"` | Indicates to search engine crawlers how frequently the content at this URL is expected to change. |
| `priority` | `number` | `1` | The priority of this URL relative to other URLs on the site (valid range: `0.0` to `1.0`). Set to highest priority (`1`). |

---

## Execution Flow

1. **Invocation**: Next.js automatically detects and invokes the default export function from `src/app/sitemap.ts` when a request is made for the `/sitemap.xml` route.
2. **Timestamp Generation**: The function initializes `lastModified` with the current system date and time.
3. **Array Construction**: It constructs an array containing one object representing the root site path (`${SITE_URL}/`).
4. **Output Generation**: Next.js converts the returned array into a formatted XML sitemap response.