# Technical Documentation: `src/openhiggsfield/key-modal.tsx`

The `KeyModal` component is a Next.js client component (`"use client"`) that renders an interactive modal dialog allowing users to enter, save, replace, or remove their platform API key credentials.

---

## Overview

- **File Path:** `src/openhiggsfield/key-modal.tsx`
- **Module Type:** Client Component
- **Primary Function:** Provides an HTML `<dialog>` interface for managing authentication API keys by interacting with platform credential actions (`savePlatformCredentials` and `clearPlatformCredentials`).

---

## Dependencies

- **React:** `useEffect`, `useRef`, `useState`, `FormEvent`
- **Actions (`@/generation/actions`):**
  - `savePlatformCredentials`: Asynchronous server action that stores the API key.
  - `clearPlatformCredentials`: Asynchronous server action that removes stored credentials.
- **Icons (`./icons`):**
  - `CloseIcon`: Icon component used in the modal close button.

---

## Component Props

The `KeyModal` component accepts a single object containing four required properties:

```typescript
{
  configured: boolean;
  onClose: () => void;
  onSaved: () => void;
  onCleared: () => void;
}
```

| Prop | Type | Description |
| :--- | :--- | :--- |
| `configured` | `boolean` | Indicates whether an API key is currently saved/configured. Determines dynamic UI text and whether the "Remove key" button is displayed. |
| `onClose` | `() => void` | Callback function executed when closing the modal (via backdrop click, close button, or standard dialog event). |
| `onSaved` | `() => void` | Callback function executed after successfully saving or updating the API key credentials. |
| `onCleared` | `() => void` | Callback function executed after successfully clearing the existing API key credentials. |

---

## Internal State & Refs

### State Variables

- **`apiKey` (`string`)**: Holds the current input value for the API key text field. Defaults to `""`.
- **`busy` (`boolean`)**: Indicates whether an asynchronous operation (`onSubmit` or `onClear`) is currently in progress. Defaults to `false`.
- **`error` (`string | null`)**: Holds any error message resulting from a failed save or clear operation. Defaults to `null`.

### References (`useRef`)

- **`ref` (`useRef<HTMLDialogElement>(null)`)**: Points to the top-level HTML `<dialog>` element. Used to invoke `.showModal()`.
- **`panelRef` (`useRef<HTMLDivElement>(null)`)**: Points to the inner container element (`ohf-dialog-panel`). Used to programmatically focus the dialog content upon mounting.

---

## Component Lifecycle & Behavior

### 1. Initialization (`useEffect`)
Upon mount, an effect runs once:
1. Calls `showModal()` on the native `<dialog>` element via `ref.current`.
2. Sets focus to the inner panel container via `panelRef.current?.focus()`.

### 2. Dialog Closing & Backdrop Clicks
- The `<dialog>` element listens to the native `onClose` event and triggers the `onClose` prop.
- An `onClick` handler on the dialog checks if the user clicked directly on the `<dialog>` element (`event.target === ref.current`). If so (indicating a backdrop click), it triggers `onClose()`.

### 3. Saving Credentials (`onSubmit`)
Triggered when submitting the API key form:
1. Calls `event.preventDefault()`.
2. Sets `busy` to `true` and resets `error` to `null`.
3. Calls `savePlatformCredentials({ api_key: apiKey })`.
4. On success: Calls `onSaved()`.
5. On failure: Catches the error and sets the `error` state (extracts `caught.message` if it is an `Error` instance, or falls back to `"Could not save the key"`).
6. In all cases: Resets `busy` to `false` in the `finally` block.

### 4. Clearing Credentials (`onClear`)
Triggered when the user clicks the "Remove key" button:
1. Sets `busy` to `true` and resets `error` to `null`.
2. Calls `clearPlatformCredentials()`.
3. Resets local `apiKey` state to `""`.
4. Calls `onCleared()`.
5. On failure: Catches the error and sets the `error` state (extracts `caught.message` if it is an `Error` instance, or falls back to `"Could not remove the key"`).
6. In all cases: Resets `busy` to `false` in the `finally` block.

---

## Component Layout & Rendering Logic

```
<dialog>
  └── <div className="ohf-dialog-panel ohf-keys-panel">
        ├── <div className="ohf-keys-head">
        │     ├── Title & Conditional Description Copy
        │     └── Close Button (<CloseIcon />)
        └── <form className="ohf-keys-form">
              ├── Password Input Field ("api_key")
              ├── [Optional] Error Alert Box
              └── Action Buttons Container
                    ├── [Optional] "Remove key" Button (if `configured` is true)
                    └── Save / Replace Submit Button
```

### Conditional Logic in UI

1. **Header Description:**
   - If `configured` is `true`: Displays `"A key is saved in this browser. Enter a new id:secret pair to replace it."`
   - If `configured` is `false`: Displays `"Paste your platform key as id:secret. It stays in an httpOnly cookie and is sent as Authorization: Key id:secret."`

2. **Error Alert:**
   - Rendered (`<div className="ohf-alert" role="alert">`) only if `error` is non-null.

3. **"Remove key" Button:**
   - Rendered only if `configured` is `true`.
   - Disabled when `busy` is `true`.

4. **Submit Button:**
   - Disabled when `busy` is `true` OR `apiKey.trim()` is empty.
   - Button text logic:
     - If `busy` is `true`: Displays `"Saving…"`
     - If `busy` is `false` and `configured` is `true`: Displays `"Replace key"`
     - If `busy` is `false` and `configured` is `false`: Displays `"Save key"`