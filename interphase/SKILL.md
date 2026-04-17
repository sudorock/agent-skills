---
name: interphase
description: >-
  Use when building React state with `@aktopia/interphase` -- creating
  registries, stores, subscriptions, events, or using `useSub`, `useEvent`,
  `suspensify` hooks and utilities.
---

# Interphase

Reactive state management for React that unifies sync and async state through subscriptions and events, built on Zustand, TanStack Query, and Immer.

## Public API

### Setup

- `createRegistry<State, Subs, Events>()` — returns `{ sub, asyncSub, event }`
- `createStore<State, Subs, Events>({ slices, registry })` — returns `{ StoreProvider, useSub, useEvent, useBoundEvent }`
- `suspensify<P>(Component)` — HOC: wraps in Suspense, adds optional `fallback` prop, memoized

### Registry Methods

- `sub(id, resolver)` — register sync subscription. Resolver: `({ state, params }) => result`
- `asyncSub(id, resolver)` — register async subscription. Resolver: `async ({ state, params }) => Promise<result>`
- `event(id, handler)` — register event handler

### Hooks

- `useSub(id, params?)` — read any subscription; suspends for async
- `useEvent(id)` — returns stable dispatcher `(params) => void | Promise<void>`
- `useBoundEvent(id, params)` — memoized zero-arg callback with bound params

### Provider

- `StoreProvider` — wraps subtree to provide store context; must wrap all hook consumers

## Type System

- `Sub<Params, Result>` — subscription type definition
- `Event<Params>` — event type definition
- `Slice<State>` — `() => Partial<State>`

## Naming Conventions

- Treat Interphase IDs as Clojure-style `namespace/name`
- The namespace may contain dots or hyphens; `/` appears once and only separates namespace from leaf name
- Use nouns for subscriptions and async subscriptions: `'current.todos/filter'`, `'todos/list'`
- Use verbs for events: `'current.todos.filter/set'`, `'todos/create'`
- If you name slice state in prose, keep the same shape: `'current.todos/state'`
- Do not use path-style IDs like `'item/filters/sort-by'`
- If you feel tempted to add a second `/`, collapse the parent path into the namespace with dots instead

## Subscription Resolver Context

Available inside `sub` and `asyncSub` resolvers:

- `state` — current store state
- `params` — subscription parameters

## Event Handler Context

Available inside `event` handlers:

- `params` — event parameters
- `setState(updater)` — Immer draft mutator
- `getState()` — current state snapshot
- `fetchSub(id, params?)` — read current subscription value
- `dispatchEvent(id, params?)` — dispatch other events
- `invalidateAsyncSub([id, params])` — refetch single async subscription
- `invalidateAsyncSubs([[id, params], ...])` — refetch multiple async subscriptions
- `replaceAsyncSub([id, params], updater)` — direct cache update without refetch
- `replaceAsyncSubs([[[id, params], updater], ...])` — batch cache updates

## Workflow

1. Define `State`, `Subs`, and `Events` types using `Sub` and `Event`
2. `createRegistry<State, Subs, Events>()`
3. Register sync reads with `sub`
4. Register async reads with `asyncSub`
5. Register writes and orchestration with `event`
6. `createStore({ registry, slices })`
7. Export `StoreProvider`, `useSub`, `useEvent`, `useBoundEvent`
8. Read state through `useSub`; wrap async consumers with `suspensify`

## Canonical Store Module

```ts
import {
  createRegistry,
  createStore,
  suspensify,
  type Event,
  type Sub,
} from '@aktopia/interphase';

type Todo = { id: string; title: string; completed: boolean };
type State = { filter: 'all' | 'active' | 'completed' };

type Subs = {
  'current.todos/filter': Sub<{}, State['filter']>;
  'todos/list': Sub<{ filter: State['filter'] }, Todo[]>;
};

type Events = {
  'current.todos.filter/set': Event<{ value: State['filter'] }>;
  'todos/create': Event<{ title: string }>;
};

const registry = createRegistry<State, Subs, Events>();
const { sub, asyncSub, event } = registry;

sub('current.todos/filter', ({ state }) => state.filter);

asyncSub('todos/list', async ({ params }) => {
  const res = await fetch(`/api/todos?filter=${params.filter}`);
  return res.json();
});

event('current.todos.filter/set', ({ params, setState }) => {
  setState((s) => { s.filter = params.value; });
});

event('todos/create', async ({ params, fetchSub, invalidateAsyncSub }) => {
  await fetch('/api/todos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: params.title }),
  });
  await invalidateAsyncSub([
    'todos/list',
    { filter: fetchSub('current.todos/filter') },
  ]);
});

const store = createStore<State, Subs, Events>({
  registry,
  slices: [() => ({ filter: 'all' })],
});

export const { StoreProvider, useSub, useEvent, useBoundEvent } = store;
```

## `suspensify`

Any component that calls `useSub` on an async subscription will suspend. Wrap these components with `suspensify` to provide Suspense boundaries. The wrapped component accepts an optional `fallback` prop for loading UI.

```tsx
export const TodoList = suspensify(function TodoList() {
  const filter = useSub('current.todos/filter');
  const todos = useSub('todos/list', { filter });

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
});

// Usage
<StoreProvider>
  <TodoList fallback={<p>Loading...</p>} />
</StoreProvider>
```

## Rules

- One store module per app: owns registry, slices, exported hooks
- Slice state is plain data; initial values go in `slices`, not components
- Derived reads belong in subscriptions, not inline in components
- Writes, async orchestration, cache invalidation, and cross-sub coordination belong in events
- `setState` is an Immer draft mutator — mutate the draft, do not return new state
- Use `fetchSub` inside events when the write depends on current derived state
- Use `dispatchEvent` inside events to compose event chains
- After remote writes, prefer `invalidateAsyncSub`/`invalidateAsyncSubs` to refetch
- Use `replaceAsyncSub` only for intentional optimistic or direct cache updates
- Use `useBoundEvent` only for fixed-parameter callbacks needing a zero-arg handler
- Use one Clojure-style `namespace/name` ID per read or action; `/` is not a path separator
- Name subscriptions as nouns (reads), events as verbs (actions)
