# Interphase

Reactive state management for React using `@aktopia/interphase`. Unifies sync and async state through subscriptions and events, built on Zustand, TanStack Query, and Immer.

## Install

```
npx skills add sudorock/agent-skills --skill interphase
```

## What It Does

Teaches the agent the full `@aktopia/interphase` API: registries, stores, subscriptions (sync and async), events, and hooks. The agent can scaffold complete store modules, wire up subscriptions, and write event handlers following the library's conventions.

### Key Concepts

- **Subscriptions** (`sub`, `asyncSub`) — declarative reads from state or async sources
- **Events** (`event`) — writes, orchestration, cache invalidation
- **Hooks** (`useSub`, `useEvent`, `useBoundEvent`) — React bindings
- **IDs** — Clojure-style `namespace/name` (nouns for reads, verbs for actions)
- **`suspensify`** — HOC wrapping async consumers in Suspense boundaries

### Example

```ts
sub('current.todos/filter', ({ state }) => state.filter);

asyncSub('todos/list', async ({ params }) => {
  const res = await fetch(`/api/todos?filter=${params.filter}`);
  return res.json();
});

event('current.todos.filter/set', ({ params, setState }) => {
  setState((s) => { s.filter = params.value; });
});
```

## System Requirements

| Dependency | Purpose |
|---|---|
| [`@aktopia/interphase`](https://www.npmjs.com/package/@aktopia/interphase) | The library this skill targets |
| React 18+ | Peer dependency |
