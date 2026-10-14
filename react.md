# ⚛️ React.js Comprehensive Interview Guide

A deep-dive, production-grade guide covering React architecture, Virtual DOM, Fiber engine, hooks internals, state management, concurrency, and performance optimization.

---

## 📑 Table of Contents
1. [React Architecture & Core Internals](#1-react-architecture--core-internals)
2. [Virtual DOM & Reconciliation (Fiber Engine)](#2-virtual-dom--reconciliation-fiber-engine)
3. [Component Lifecycle & Render Phases](#3-component-lifecycle--render-phases)
4. [Hooks Deep Dive & Internals](#4-hooks-deep-dive--internals)
5. [React 18 & 19 Concurrency Features](#5-react-18--19-concurrency-features)
6. [State Management Ecosystem](#6-state-management-ecosystem)
7. [React Performance Optimization](#7-react-performance-optimization)
8. [Advanced Patterns & Component Architecture](#8-advanced-patterns--component-architecture)
9. [Server Components (RSC) & Modern Data Fetching](#9-server-components-rsc--modern-data-fetching)
10. [Common React Pitfalls & Interview Questions](#10-common-react-pitfalls--interview-questions)

---

## 1. React Architecture & Core Internals

### What is JSX?
- [ ] JSX is syntax sugar for `React.createElement(type, props, ...children)` (or the modern JSX runtime `jsx(type, props)`).
- [ ] Transpiled at build time by Babel, SWC, or esbuild into plain JavaScript object representations called **React Elements**.
- [ ] React Elements are plain JS objects: `{ $$typeof: Symbol(react.element), type, props, key, ref }`. The `$$typeof` symbol protects against cross-site scripting (XSS) injecting arbitrary JSON as HTML.

### Declarative vs Imperative UI
- [ ] *Imperative* (jQuery / Vanilla DOM): Step-by-step instructions telling the browser *how* to mutate nodes (`element.appendChild()`, `element.classList.add()`).
- [ ] *Declarative* (React): You describe *what* the UI should look like for a given state $UI = f(state)$. React handles the low-level DOM mutations.

---

## 2. Virtual DOM & Reconciliation (Fiber Engine)

### The Problem with Real DOM Updates
- [ ] Real DOM tree mutations are expensive because browser calculations trigger layout (reflow) and repaint across the document.
- [ ] The Virtual DOM is an in-memory tree of JS objects representing the UI.

### Reconciliation & Heuristic Diffing Algorithm
The diffing algorithm operates in $O(n)$ time using two key assumptions:
1. **Different element types generate different trees**: If a `<div>` changes to a `<section>`, React completely destroys the old subtree and builds a new one from scratch.
2. **Keys indicate child stability**: Keys must be unique among siblings and stable across renders.
   - *Why NOT use index as key?* If items are inserted, deleted, or sorted, index-based keys cause React to preserve the wrong component state and re-render unnecessarily.

### React Fiber Architecture (React 16+)
- [ ] **Stack Reconciler (Old)**: Synchronous recursion. Once a render started, it blocked the main thread until the entire tree was reconciled, causing frame drops and input lag.
- [ ] **Fiber Reconciler**:
  - Re-implementation of the reconciler using a virtual stack frame structure.
  - Each Fiber is a JS object representing a work unit:
    ```typescript
    type Fiber = {
      tag: WorkTag;
      type: any;
      stateNode: any; // Real DOM node or class instance
      return: Fiber | null; // Parent fiber
      child: Fiber | null;  // First child fiber
      sibling: Fiber | null;// Next sibling fiber
      memoizedState: any;  // Hook linked list or component state
      memoizedProps: any;
      lanes: Lanes;        // Priority lanes
      alternate: Fiber | null; // Double-buffering partner (current <-> workInProgress)
    };
    ```
- [ ] **Double Buffering Pattern**:
  - `current` tree: What is currently rendered on screen.
  - `workInProgress` (WIP) tree: Tree being built in the background during render phase.
  - Once rendering finishes, React simply swaps the root pointer (`current = workInProgress`).

---

## 3. Component Lifecycle & Render Phases

### The Two Major Phases
```
┌────────────────────────────────────────────────────────┐
│ 1. Render Phase (Reconciliation)                       │
│    - Pure, no DOM side-effects                         │
│    - Can be paused, aborted, or restarted by React     │
│    - Calls component functions, evaluates JSX, diffs   │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ 2. Commit Phase                                        │
│    - Synchronous, cannot be interrupted                │
│    - Updates real DOM nodes                            │
│    - Runs useLayoutEffect synchronously                │
│    - Browser repaints the screen                       │
│    - Runs useEffect asynchronously                     │
└────────────────────────────────────────────────────────┘
```

---

## 4. Hooks Deep Dive & Internals

### How Hooks Work Internally
- [ ] Hooks are stored as a **singly-linked list** on the Fiber's `memoizedState` property.
- [ ] Each hook node has: `{ memoizedState, next, queue }`.
- [ ] **Why Rules of Hooks exist**:
  - React walks the linked list in the exact order hooks are declared on every render.
  - If a hook is placed inside an `if` condition or loop, the index/order of subsequent hooks shifts, causing memory corruption or reading the wrong hook's state.

### Core Hooks Comparison

#### `useState` vs `useReducer`
- `useState`: Best for independent, primitive state values.
- `useReducer`: Best for complex state objects with multiple related transitions or state logic that depends on previous states.
- *Functional updater*: Always use `setCount(prev => prev + 1)` when the next state depends on the previous state to avoid race conditions.

#### `useEffect` vs `useLayoutEffect`
| Feature | `useEffect` | `useLayoutEffect` |
| :--- | :--- | :--- |
| **Execution Timing** | Asynchronous, **after** browser paint | Synchronous, **before** browser paint |
| **Blocks Paint?** | No | Yes |
| **Best For** | Data fetching, subscriptions, logging | Measuring DOM elements (e.g. tooltip position), avoiding visual flickers |

#### `useMemo` vs `useCallback`
- `useMemo(() => computeValue(a, b), [a, b])`: Caches the **result** of a calculation.
- `useCallback(fn, deps)`: Caches the **function instance** across renders. Equivalent to `useMemo(() => fn, deps)`.
- *When to use*: Only when passing callbacks to children wrapped in `React.memo`, or when passing objects/functions as dependencies to other hooks.

#### `useRef`
- Stores a mutable reference that persists across all renders.
- Mutating `.current` does **not** trigger a re-render.
- Primary uses: Storing DOM references, storing timer/interval IDs, keeping track of previous state.

---

## 5. React 18 & 19 Concurrency Features

### Automatic Batching
- In React 17 and earlier, state updates inside `setTimeout`, `Promise`, or native event handlers were not batched.
- React 18 batches **all** state updates automatically regardless of where they originate:
  ```javascript
  setTimeout(() => {
    setCount(c => c + 1);
    setFlag(f => !f);
    // React 18 triggers only ONE re-render!
  }, 1000);
  ```

### Concurrent Transitions: `useTransition` & `useDeferredValue`
- [ ] **`useTransition`**:
  ```javascript
  const [isPending, startTransition] = useTransition();

  function handleSearch(e) {
    const query = e.target.value;
    setInputValue(query); // Urgent update: user typing input

    startTransition(() => {
      setSearchQuery(query); // Non-urgent update: filtering 10,000 items
    });
  }
  ```
- [ ] **`useDeferredValue`**: Defers updating a value until urgent renders finish, keeping the UI responsive.

### `Suspense` for Data Fetching
- Allows components to "suspend" rendering while waiting for an asynchronous operation to resolve, showing a `fallback` UI in the meantime.

---

## 6. State Management Ecosystem

### Comparison Matrix
| Solution | Type | Ideal Use Case | Pros | Cons |
| :--- | :--- | :--- | :--- | :--- |
| **React Context** | Built-in | Low-frequency global state (theme, auth user, language) | No extra bundle | Re-renders all consumers unless split carefully |
| **Redux Toolkit (RTK)** | Flux / Centralized | Large-scale apps with complex transactional state | Predictable, DevTools, time-travel debugging | Boilerplate for small apps |
| **Zustand** | Store-based (Hooks) | Medium-to-large apps | Minimal boilerplate, no Context provider required | Less strict conventions |
| **TanStack Query (React Query)** | Server State | API caching, pagination, deduplication, sync | Eliminates 80% of client state code | Not for purely local UI state |

---

## 7. React Performance Optimization

### 1. Identifying Unnecessary Re-renders
- [ ] A component re-renders when:
  1. Its state changes.
  2. Its parent re-renders (even if props did not change!).
  3. A context it subscribes to changes.
- [ ] Use **React DevTools Profiler** to record and inspect render durations and reasons.

### 2. Preventing Re-renders
- [ ] **Component Composition** (Lift content up):
  ```jsx
  // Instead of re-rendering HeavyComponent when count changes:
  function CounterWithChildren({ children }) {
    const [count, setCount] = useState(0);
    return (
      <div>
        <button onClick={() => setCount(c => c + 1)}>{count}</button>
        {children} {/* Passed as prop: NOT re-rendered! */}
      </div>
    );
  }
  ```
- [ ] `React.memo(Component, arePropsEqual)`: Skips rendering if shallow comparison of props is identical.
- [ ] Split large Contexts into separate domain contexts (e.g., `ThemeContext`, `AuthContext`) or separate state from dispatchers.

### 3. Code Splitting & Lazy Loading
```jsx
import { lazy, Suspense } from 'react';

const HeavyDashboard = lazy(() => import('./HeavyDashboard'));

function App() {
  return (
    <Suspense fallback={<div>Loading dashboard...</div>}>
      <HeavyDashboard />
    </Suspense>
  );
}
```

---

## 8. Advanced Patterns & Component Architecture

### Compound Components Pattern
```jsx
// Usage:
<Tabs defaultTab="tab1">
  <Tabs.List>
    <Tabs.Trigger id="tab1">Profile</Tabs.Trigger>
    <Tabs.Trigger id="tab2">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel id="tab1"><ProfileView /></Tabs.Panel>
  <Tabs.Panel id="tab2"><SettingsView /></Tabs.Panel>
</Tabs>
```
*Implemented using React Context internally to share active tab state without explicit prop-drilling.*

### Custom Hook: `useDebounce`
```typescript
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

---

## 9. Server Components (RSC) & Modern Data Fetching

- [ ] **Server Components (`.server.jsx` or default in Next.js App Router)**:
  - Run only on the server.
  - Never ship JavaScript to the client (Zero bundle size impact).
  - Can directly query databases and file systems.
  - Cannot use `useState`, `useEffect`, or browser APIs.
- [ ] **Client Components (`'use client'`)**:
  - Rendered on the server initially (SSR) and hydrated on the client.
  - Have access to interactivity, state, event listeners, and browser APIs.

---

## 10. Common React Pitfalls & Interview Questions

### 1. Stale Closures in `useEffect`
```javascript
// BUG: count is locked to 0
useEffect(() => {
  const id = setInterval(() => {
    console.log(count); // Always 0!
  }, 1000);
  return () => clearInterval(id);
}, []); // Empty dependencies!

// FIX: Include count in deps OR use functional updater in state setter
```

### 2. Mutating State Directly
```javascript
// WRONG: React does not detect mutation
user.name = 'New Name';
setUser(user); // Object.is(user, user) === true! No re-render!

// CORRECT:
setUser({ ...user, name: 'New Name' });
```

### 3. Derived State Anti-pattern
Do not duplicate props in state:
```javascript
// BAD:
const [fullName, setFullName] = useState(`${props.first} ${props.last}`);

// GOOD: Compute during render
const fullName = `${props.first} ${props.last}`;
```
