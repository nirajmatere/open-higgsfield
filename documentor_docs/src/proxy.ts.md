# Documentation: `src/proxy.ts`

## Overview

The `src/proxy.ts` module handles device identification cookies on incoming HTTP requests. It uses Next.js server components (`NextRequest` and `NextResponse`) to check for an existing device cookie, resolve or generate a device ID via an external utility, and set a response cookie if a new device ID was minted. It also exports a configuration object specifying which routes this logic applies to.

---

## Dependencies

*   **`NextResponse`**, **`NextRequest`** (from `"next/server"`): Used for handling and manipulating Next.js HTTP server requests and responses, specifically for reading and writing cookies and continuing request processing.
*   **`DEVICE_COOKIE`**, **`DEVICE_COOKIE_OPTIONS`**, **`resolveDeviceId`** (from `"./generation/device"`):
    *   `DEVICE_COOKIE`: The name/key of the device cookie.
    *   `DEVICE_COOKIE_OPTIONS`: Configuration options applied when setting the device cookie (e.g., domain, path, maxAge).
    *   `resolveDeviceId`: Function that accepts a cookie value string (or `undefined`) and returns an object containing `{ deviceId, minted }`.

---

## Functions

### `proxy(request: NextRequest)`

Primary handler function that processes incoming requests to ensure device identification.

#### Signature
```typescript
export function proxy(request: NextRequest): NextResponse
```

#### Detailed Execution Flow
1. **Retrieve Cookie Value**: Extracts the string value of the cookie specified by `DEVICE_COOKIE` from `request.cookies`.
2. **Resolve Device ID**: Passes the cookie value into `resolveDeviceId(...)`, which returns an object with two properties:
   * `deviceId`: The resolved or generated device identifier.
   * `minted`: A boolean indicating whether a new device ID was created (`true`) or an existing one was used (`false`).
3. **Check `minted` Status**:
   * If `minted` is falsy (`false`), no cookie update is needed. The function returns `NextResponse.next()`, allowing the request to proceed unchanged.
   * If `minted` is truthy (`true`), a new response is created using `NextResponse.next()`.
4. **Set Cookie**: Attaches the new `DEVICE_COOKIE` to the response with value `deviceId` and options `DEVICE_COOKIE_OPTIONS`.
5. **Return Response**: Returns the modified `NextResponse` object containing the set cookie.

---

## Configuration

### `config`

An exported configuration object that controls route matching.

```typescript
export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp|ico)$).*)"],
};
```

#### Matcher Behavior
The `matcher` array uses a negative lookahead regular expression to run the handler on all incoming request paths **except**:
* `/api/*` (API routes)
* `/_next/static/*` (Static assets)
* `/_next/image/*` (Optimized image assets)
* `/favicon.ico`
* Static media files with extensions: `.svg`, `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.ico`