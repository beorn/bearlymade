# alien-resources

Async signal bridge for [alien-signals](https://github.com/stackblitz/alien-signals). Wraps async fetchers into reactive signals with loading/error states and automatic cancellation.

## Install

```bash
npm install alien-resources alien-signals
```

`alien-signals` is a peer dependency.

## Usage

```typescript
import { signal, effect } from "alien-signals"
import { createResource } from "alien-resources"

const userId = signal("user-1")

// Resource tracks a fetcher — refetches when dependencies change
const profile = createResource(async (abort) => {
  const res = await fetch(`/api/users/${userId()}`, { signal: abort })
  return res.json()
})

// Read data, loading, and error reactively
effect(() => {
  if (profile.loading()) console.log("Loading...")
  else if (profile.error()) console.log("Error:", profile.error())
  else console.log("Data:", profile())
})

// Change dependency — automatically refetches (cancels in-flight request)
userId("user-2")
```

## API

### `createResource(fetcher, options?)`

Creates a reactive resource that tracks an async fetcher.

**Parameters:**

- `fetcher: (abort: AbortSignal) => Promise<T>` — Async function that produces data. Receives an `AbortSignal` for cancellation. Any alien-signals signals read in the synchronous preamble (before the first `await`) become reactive dependencies.
- `options.initialValue?: T` — Initial data value (default: `undefined`).

**Returns: `Resource<T>`**

| Member | Type | Description |
|--------|------|-------------|
| `resource()` | `T \| undefined` | Read current data. `undefined` while loading (unless `initialValue` set). |
| `resource.loading()` | `boolean` | Whether a fetch is in progress. |
| `resource.error()` | `Error \| null` | Error from last fetch, or `null`. |
| `resource.refetch()` | `void` | Manually trigger a re-fetch. |
| `resource.mutate(value)` | `void` | Optimistically set data (overwritten on next fetch). |
| `resource.dispose()` | `void` | Stop tracking, cancel in-flight fetches. |

## How It Works

1. The fetcher runs inside an `effect()` tracking context.
2. Any signals read during the fetcher's synchronous portion (before the first `await`) become dependencies.
3. When dependencies change, the effect re-runs — aborting any in-flight request via `AbortController` and starting a new fetch.
4. A monotonic fetch ID ensures only the latest fetch writes to the data/loading/error signals (stale responses are discarded).

## Examples

### Initial value

```typescript
const data = createResource(
  async () => fetchExpensiveData(),
  { initialValue: [] }
)
// data() returns [] immediately, then the fetched result
```

### Manual refetch

```typescript
const feed = createResource(async () => fetchFeed())

// Poll every 30 seconds
setInterval(() => feed.refetch(), 30_000)
```

### Optimistic update

```typescript
const todos = createResource(async () => fetchTodos())

function addTodo(todo: Todo) {
  // Show immediately
  todos.mutate([...todos()!, todo])
  // Persist (refetch will overwrite with server state)
  postTodo(todo).then(() => todos.refetch())
}
```

### Cleanup

```typescript
const resource = createResource(async (abort) => {
  const res = await fetch(url, { signal: abort })
  return res.json()
})

// When done — stops tracking dependencies, cancels in-flight fetch
resource.dispose()
```

## License

MIT
