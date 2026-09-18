# Technical Documentation: `src/app/robots.ts`

## Overview

The `src/app/robots.ts` file is a Next.js dynamic metadata route that programmatically generates a `robots.txt` file. This file controls web crawler behavior by defining access rules (allow/disallow directives), specifying the location of the site's XML sitemap, and setting the preferred host URL.

---

## Imports

```typescript
import type { MetadataRoute } from "next";
import { SITE_URL } from "@/site";
```

* **`type { MetadataRoute }`**: Imported from `next`. Provides TypeScript types (`MetadataRoute.Robots`) to ensure the return object matches Next.js's expected structure for generating `robots.txt`.
* **`SITE_URL`**: Imported from the local module `@/site`. Represents the base URL string used to build absolute URLs for the host and sitemap directives.

---

## Function & Logic Breakdown

### `robots()` Function

```typescript
export default function robots(): MetadataRoute.Robots
```

The default export function returns an object typed as `MetadataRoute.Robots`. Next.js automatically calls this function to generate the site's `robots.txt` output.

### Configuration Object

The returned object contains three top-level keys: `rules`, `sitemap`, and `host`.

```typescript
return {
  rules: {
    userAgent: "*",
    allow: "/",
    disallow: ["/api/"],
  },
  sitemap: `${SITE_URL}/sitemap.xml`,
  host: SITE_URL,
};
```

#### 1. `rules`
Defines crawling permissions for search engine spiders and web scrapers.

* **`userAgent: "*"`**: Applies the specified crawling rules to all web crawlers.
* **`allow: "/"`**: Explicitly grants search engines permission to crawl all pages starting from the root directory (`/`).
* **`disallow: ["/api/"]`**: Prevents web crawlers from indexing or accessing any endpoints under the `/api/` path.
  * *Code Comment Context*: The inline comment notes that API endpoints such as `/api/blob` issue upload tokens and are not intended to appear as search results.

#### 2. `sitemap`
* Value: `${SITE_URL}/sitemap.xml`
* Specifies the full absolute URL pointing to the site's XML sitemap file.

#### 3. `host`
* Value: `SITE_URL`
* Specifies the site's primary host URL.

---

## Summary of Return Structure

| Field | Type | Description |
| :--- | :--- | :--- |
| `rules.userAgent` | `string` | Applies rules to all crawlers (`"*"`) |
| `rules.allow` | `string` | Allows access to root directory (`"/"`) |
| `rules.disallow` | `string[]` | Restricts access to API routes (`["/api/"]`) |
| `sitemap` | `string` | Absolute URL to `sitemap.xml` using `SITE_URL` |
| `host` | `string` | The canonical domain set via `SITE_URL` |