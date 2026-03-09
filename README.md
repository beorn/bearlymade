# vitest-silvery-dots

Streaming dot reporter for [Vitest](https://vitest.dev/), built with [Silvery](https://silvery.dev) React terminal UI.

## Install

```bash
bun add -d vitest-silvery-dots
```

## Usage

```bash
vitest --reporter=vitest-silvery-dots
```

Or in `vitest.config.ts`:

```ts
export default defineConfig({
  test: {
    reporters: ['vitest-silvery-dots'],
  },
})
```

## Part of [Bearly Made](https://github.com/beorn/bearlymade)

Small dev tools by [Beorn](https://beorn.codes).
