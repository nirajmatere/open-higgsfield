# Module Documentation: `src/generation/poll.ts`

## Overview

The `src/generation/poll.ts` module provides a client-side batching and polling mechanism to track the progress of asynchronous generation requests until they reach a terminal status (e.g., completed, failed, nsfw, or canceled).

Instead of spawning individual polling timers or separate server actions for each active request, this module aggregates all active request IDs into a single consolidated polling loop (`round()`). This prevents queuing bottlenecks when using single-threaded or sequential server action execution models.

---

## Constants

| Constant | Type | Value | Description |
| :--- | :--- | :--- | :--- |
| `TERMINAL` | `Set<string>` | `new Set(["completed", "failed", "nsfw", "canceled"])` | Set of statuses that indicate a generation job has reached a final state and will not change again. |
| `POLL_INTERVAL_MS` | `number` | `4000` (4 seconds) | Time delay between batch polling attempts. |
| `POLL_DEADLINE_MS` | `number` | `600000` (10 minutes) | Default maximum duration a request can remain pending before timing out. |
| `MAX_MISSES` | `number` | `3` | Maximum allowed consecutive polling errors before all pending requests are rejected. |

---

## Internal State & Types

### Types

#### `Waiter`
Internal representation of a monitored request's state and promise handlers:
```typescript
type Waiter = {
  deadline: number;
  resolve: (status: GenerationStatus) => void;
  reject: (reason: Error) => void;
};
```

### Module State Variables

*   `waiting`: `Map<string, Waiter>` – Tracks active request IDs mapped to their deadline and resolution/rejection callbacks.
*   `inflight`: `Map<string, Promise<GenerationStatus>>` – Deduplication map storing existing promises for currently monitored requests.
*   `timer`: `ReturnType<typeof setTimeout> | null` – Stores the handle for the active polling delay timer.
*   `polling`: `boolean` – Flag indicating whether a polling request (`round()`) is currently in progress.
*   `misses`: `number` – Counter tracking consecutive batch polling failures.

---

## Exported Functions

### `watchRequest(requestId, opts)`

Monitors a specific generation request until it resolves to a terminal state or fails.

```typescript
export function watchRequest(
  requestId: string,
  opts?: { deadline?: number },
): Promise<GenerationStatus>
```

*   **Parameters:**
    *   `requestId` (`string`): The unique identifier of the generation request to track.
    *   `opts` (`{ deadline?: number }`, optional): Configuration options. `deadline` specifies an absolute timestamp (in milliseconds) when the request should time out.
*   **Returns:** `Promise<GenerationStatus>` – Resolves with the `GenerationStatus` when terminal, or rejects on error/timeout.
*   **Behavior:**
    1.  **Deduplication:** Checks if `inflight` already contains a promise for `requestId`. If found, returns the existing promise.
    2.  Creates a new `Promise` and stores a `Waiter` object in `waiting`.
    3.  When the promise resolves or rejects, `inflight.delete(requestId)` is automatically executed to clean up state.
    4.  Calls `schedule()` to ensure the polling cycle is active.
    5.  Stores the promise in `inflight` and returns it.

---

### `stopWatching()`

Stops all polling activities and discards all active monitoring requests without resolving or rejecting their underlying promises.

```typescript
export function stopWatching(): void
```

*   **Behavior:**
    1.  Clears any active timeout (`timer`).
    2.  Resets `misses` counter to `0`.
    3.  Clears the `waiting` map.
    4.  Clears the `inflight` map.
*   **Use Case:** Called when the consuming component unmounts and active results are no longer required by the UI.

---

## Internal Helper Functions

### `schedule()`
```typescript
function schedule(): void
```
Schedules the next polling round using `setTimeout`. It guards against duplicate timers or unnecessary runs by aborting early if:
*   `timer` is already set (`timer !== null`).
*   A polling cycle is currently in progress (`polling === true`).
*   There are no active requests in the queue (`waiting.size === 0`).

---

### `round()`
```typescript
async function round(): Promise<void>
```
Executes a single polling iteration:
1. Resets `timer` to `null` and sets `polling = true`.
2. Collects all request IDs from `waiting.keys()` and calls `getGenerationStatuses({ requestIds })`.
3. **On Success:** Resets `misses` to `0`, passes each `StatusResult` to `deliver()`, and calls `sweep()` to clean up expired requests.
4. **On Error:** Increments `misses`. If `misses` reaches or exceeds `MAX_MISSES` (`3`), calls `settleAll()` with the caught error to reject all waiters.
5. **Finally:** Sets `polling = false` and calls `schedule()` to trigger the next interval if waiters remain.

---

### `deliver(result)`
```typescript
function deliver(result: StatusResult): void
```
Processes an individual status response for a request:
1. Retrieves the corresponding `Waiter` from `waiting`. If none exists, execution stops.
2. If `result` contains an `error` property, removes the ID from `waiting` and rejects the waiter with `new Error(result.error)`.
3. If `result.status.status` is NOT in the `TERMINAL` set, leaves the waiter in `waiting` to continue polling.
4. If `result.status.status` IS terminal, removes the ID from `waiting` and resolves the waiter with `result.status`.

---

### `sweep()`
```typescript
function sweep(): void
```
Checks all entries in `waiting` against their individual deadline timestamps:
*   Iterates through all entries in `waiting`.
*   If `Date.now() > waiter.deadline`, removes the request from `waiting` and rejects the waiter's promise with `new Error("timed out waiting for the platform")`.

---

### `settleAll(reason)`
```typescript
function settleAll(reason: Error): void
```
Clears the queue and rejects all currently waiting requests with a given error:
1. Copies all waiters from `waiting`.
2. Clears `waiting` map and resets `misses` to `0`.
3. Iterates over the cached waiters and invokes `reject(reason)` for each.

---

## Execution Flow Summary

```
 watchRequest(id)
       │
       ▼
 [Is request in `inflight`?] ──────► Yes ─► Return existing Promise
       │
       No
       │
       ▼
 Add to `waiting` & `inflight`
       │
       ▼
  schedule() ─────► [Timer active or polling?] ──► Yes ─► (Do nothing)
       │
       No
       │
       ▼
Wait POLL_INTERVAL_MS
       │
       ▼
    round() ────────────────────────────────────────┐
       │                                            │
       ▼                                            ▼
 getGenerationStatuses([ids])                     (Error)
       │                                            │
       ├───► Success                                ▼
       │     │                               Increment `misses`
       │     ├─► deliver(result)                    │
       │     │     └─► If Terminal/Error ──► Resolve/Reject & remove
       │     │                                      │
       │     ├─► sweep()                            ▼
       │     │     └─► If Expired ─────────► Reject & remove
       │     │                                [misses >= MAX_MISSES?]
       │     └─► misses = 0                         │
       │                                            ├─► Yes ─► settleAll(error)
       ▼                                            │
  schedule() ◄──────────────────────────────────────┴─► No ──► schedule()
```