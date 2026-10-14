# ⚛️ React.js Comprehensive Master Interview Guide

A complete, production-grade guide covering everything tested in junior, senior, and staff-level React frontend interviews — from fundamental component models to Fiber internals, every single hook, React 18/19 concurrency, performance optimization, state management, and architectural patterns.

---

## 📑 Table of Contents
1. [React Core Architecture & Fundamentals](#1-react-core-architecture--fundamentals)
2. [Component Models: Class vs Functional Components](#2-component-models-class-vs-functional-components)
3. [Virtual DOM & Reconciliation (The Fiber Engine)](#3-virtual-dom--reconciliation-the-fiber-engine)
4. [Component Lifecycle & Render Phases](#4-component-lifecycle--render-phases)
5. [The Complete React Hooks Encyclopedia (All Built-in Hooks)](#5-the-complete-react-hooks-encyclopedia)
6. [React 18 & 19 Concurrency & Next-Gen Features](#6-react-18--19-concurrency--next-gen-features)
7. [Advanced Design Patterns in React](#7-advanced-design-patterns-in-react)
8. [State Management: Context vs RTK vs Zustand vs React Query](#8-state-management-ecosystem)
9. [Performance Optimization & Profiling](#9-performance-optimization--profiling)
10. [Portals, Error Boundaries & Synthetic Events](#10-portals-error-boundaries--synthetic-events)
11. [Testing React Components (React Testing Library)](#11-testing-react-components)
12. [Top 20 Real-World React Interview Scenarios & Gotchas](#12-top-20-real-world-react-interview-scenarios--gotchas)

---

## 1. React Core Architecture & Fundamentals

### What is JSX?
- [ ] JSX is syntax sugar for `React.createElement(type, props, ...children)` (or the modern JSX runtime `jsx(type, props)`).
- [ ] Transpiled at compile time by Babel, SWC, or esbuild into plain JavaScript object representations called **React Elements**.
- [ ] React Elements are plain, immutable JS objects:
  ```javascript
  {
    $$typeof: Symbol.for('react.element'), // Security marker preventing XSS via JSON
    type: 'button',
    props: { className: 'btn', children: 'Click me' },
    key: null,
    ref: null
  }
  ```
- [ ] **Why the `$$typeof` symbol exists**: If an attacker attempts to inject a simulated JSON object as a React element from an external API, pure JSON cannot serialize `Symbol` values. React checks `element.$$typeof === Symbol.for('react.element')` and rejects untrusted objects.

### Declarative vs Imperative UI
- [ ] **Imperative (e.g. Vanilla DOM, jQuery)**: You explicitly tell the browser *how* to change the UI step-by-step (`element.appendChild()`, `element.classList.add()`, manual event binding).
- [ ] **Declarative (React)**: You specify *what* the UI should look like for a given state $UI = f(\text{state})$. React's reconciler abstracts away the underlying DOM operations.

### Controlled vs Uncontrolled Components
- [ ] **Controlled Component**: Form input whose value is completely governed by React component state (`value={state}` + `onChange={e => setState(e.target.value)}`).
  - *Pros*: Immediate validation, conditional field disabling, single source of truth.
  - *Cons*: Triggers re-renders on every keystroke.
- [ ] **Uncontrolled Component**: Form input whose value is maintained directly by the DOM itself. Access values via a `ref` (`<input ref={inputRef} defaultValue="John" />`).
  - *Pros*: Closer to vanilla HTML, fast for simple forms with no dynamic validation.

---

## 2. Component Models: Class vs Functional Components

### Lifecycle Methods Mapping to Hooks
| Class Component Lifecycle | Functional Component Hook Equivalent | Notes |
| :--- | :--- | :--- |
| `componentDidMount` | `useEffect(() => { ... }, [])` | Runs once after initial paint |
| `componentDidUpdate` | `useEffect(() => { ... }, [deps])` | Runs when any dependency changes |
| `componentWillUnmount` | `useEffect(() => () => { ... }, [])` | Cleanup function runs before unmount |
| `shouldComponentUpdate` | `React.memo(Component, arePropsEqual)` | Optimizes re-renders |
| `getDerivedStateFromProps`| Updating state directly during render (or derived state) | Anti-pattern in most cases |
| `getSnapshotBeforeUpdate`| `useLayoutEffect` | Captures DOM measurements (scroll pos) before paint |
| `componentDidCatch` | **No hook equivalent yet** | Must use Class Component for Error Boundaries |

### `PureComponent` vs `Component`
- `React.PureComponent` implements `shouldComponentUpdate()` with a **shallow comparison** of current props/state against next props/state.
- `React.Component` re-renders every time `setState()` is called or its parent re-renders, regardless of whether props/state changed.
- In functional components, `React.memo` provides the equivalent shallow prop comparison.

---

## 3. Virtual DOM & Reconciliation (The Fiber Engine)

### The Heuristic Diffing Algorithm ($O(n)$)
A generic tree comparison algorithm takes $O(n^3)$ time. React reduces this to $O(n)$ based on two practical assumptions:
1. **Different component/element types produce different subtrees**: If `<div className="box">` changes to `<span className="box">`, React unmounts the `div` and its entire subtree completely, destroying all state, and mounts a brand new `span`.
2. **Child stability is determined by Keys**:
   - Keys must be unique among siblings and stable across renders.
   - *Why NOT use array index as key?*
     - If an item is prepended or deleted from the middle, all subsequent item indices change. React matches elements by index, reusing old DOM nodes and preserving incorrect local component state (e.g. checked checkboxes or input values).

### React Fiber Engine (React 16+)
- [ ] **The Legacy Stack Reconciler**: Used recursive JavaScript function calls down the component tree. Once rendering began, it could not be paused or interrupted. Heavy trees caused noticeable frame drops and frozen inputs.
- [ ] **Fiber Reconciler**:
  - Re-implementation of the reconciler as a virtual call stack built with a linked list.
  - Each Fiber represents a **unit of work**:
    ```typescript
    interface Fiber {
      tag: WorkTag;              // FunctionComponent, ClassComponent, HostComponent, etc.
      type: any;                  // 'div', 'button', or MyComponent function
      stateNode: any;             // Reference to actual DOM node or class instance
      return: Fiber | null;       // Parent fiber
      child: Fiber | null;        // First child fiber
      sibling: Fiber | null;      // Next sibling fiber
      memoizedState: any;         // Head of the hooks linked list or component state
      memoizedProps: any;         // Props evaluated in previous render
      pendingProps: any;          // Incoming props
      lanes: Lanes;               // Bitmask representing priority level
      alternate: Fiber | null;    // WorkInProgress <-> Current counterpart (Double Buffering)
    }
    ```
- [ ] **Double Buffering Pattern**:
  - React maintains two Fiber trees at any time:
    1. **`current` tree**: Represents the nodes currently visible on the screen.
    2. **`workInProgress` (WIP) tree**: Built and mutated in memory during the Render phase.
  - When rendering finishes, React simply points the root pointer to the WIP tree (`current = workInProgress`), achieving instant, flicker-free updates.

---

## 4. Component Lifecycle & Render Phases

```
                    TRIGGER (Initial Mount / State Update / Props Change)
                                        │
┌───────────────────────────────────────▼────────────────────────────────────────┐
│ 1. RENDER PHASE (Reconciliation)                                               │
│    - Pure, synchronous or asynchronous, NO side-effects permitted!             │
│    - Calls component functions, evaluates JSX, runs diffing algorithm          │
│    - Can be paused, yielded, or restarted by the Fiber scheduler               │
└───────────────────────────────────────┬────────────────────────────────────────┘
                                        │
┌───────────────────────────────────────▼────────────────────────────────────────┐
│ 2. COMMIT PHASE (DOM Mutation)                                                 │
│    - Synchronous, uninterrupted                                                │
│    - Real DOM nodes are created, updated, or removed                           │
│    - `useLayoutEffect` runs synchronously (DOM mutated, before paint)          │
│    - Browser performs Layout (Reflow) and Paint                                │
│    - `useEffect` callbacks run asynchronously (after paint)                    │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. The Complete React Hooks Encyclopedia

Every built-in hook explained with internal mechanics, parameters, and practical rules:

### 1. `useState`
- [ ] Returns `[state, dispatch]`.
- [ ] **Functional Updates**: Always use `setCount(prev => prev + 1)` when the next value depends on the previous state to avoid race conditions and stale closures.
- [ ] **Lazy State Initialization**: If the initial state requires heavy computation, pass a function `useState(() => computeHeavy())` so it executes only on the initial mount.

### 2. `useReducer`
- [ ] `const [state, dispatch] = useReducer(reducer, initialArg, init)`.
- [ ] Decouples complex state transitions from rendering logic. Ideal when state contains multiple sub-values or when the next state depends on multiple previous values.

### 3. `useEffect`
- [ ] Runs **asynchronously after browser paint**.
- [ ] Never blocks visual rendering.
- [ ] Cleanup function executes before the component re-runs the effect or unmounts.
- [ ] *Rule*: Every reactive value (props, state, or derived variables) read inside `useEffect` must be declared in its dependency array.

### 4. `useLayoutEffect`
- [ ] Runs **synchronously after DOM mutations**, but **before the browser paints**.
- [ ] Use cases: Measuring DOM node dimensions (`getBoundingClientRect`), calculating tooltip or popover coordinates to eliminate visual flickers.
- [ ] *Caution*: Blocks the browser from painting until completed; overuse degrades INP.

### 5. `useRef`
- [ ] `const ref = useRef(initialValue)`. Returns a persistent mutable object `{ current: initialValue }`.
- [ ] Mutating `.current` does **not** trigger a component re-render.
- [ ] Common usages: Accessing DOM elements, storing timer/interval IDs, holding previous prop/state values.

### 6. `useMemo`
- [ ] `const cachedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])`.
- [ ] Caches the **result** of a calculation between renders.
- [ ] Only use when calculation is genuinely expensive (e.g. filtering 10,000 items) or preserving referential equality for child memoization.

### 7. `useCallback`
- [ ] `const cachedFn = useCallback(fn, [deps])`.
- [ ] Caches a **function instance** across renders.
- [ ] Equivalent to: `useMemo(() => fn, [deps])`.
- [ ] Useful primarily when passing callbacks to child components wrapped in `React.memo`.

### 8. `useContext`
- [ ] `const value = useContext(MyContext)`.
- [ ] Reads the nearest `<MyContext.Provider value={...}>` above in the tree.
- [ ] *Performance Warning*: Any update to the context `value` triggers a re-render in **every component that calls `useContext(MyContext)`**, regardless of whether the component uses the modified slice.

### 9. `useId` (React 18+)
- [ ] Generates unique, stable IDs across server-side rendering (SSR) and client-side hydration, preventing hydration mismatch errors.
- [ ] Ideal for accessible form associations (`htmlFor={id}` and `id={id}`).

### 10. `useImperativeHandle`
- [ ] Customizes the instance value exposed to parent components when using `ref`.
  ```jsx
  const CustomInput = forwardRef((props, ref) => {
    const inputRef = useRef();
    useImperativeHandle(ref, () => ({
      focusInput: () => inputRef.current.focus(),
      clearInput: () => { inputRef.current.value = ''; }
    }));
    return <input ref={inputRef} {...props} />;
  });
  ```

### 11. `useSyncExternalStore` (React 18+)
- [ ] Recommended primitive for subscribing to external state stores (Redux, Zustand, browser APIs like `navigator.onLine` or `window.matchMedia`).
- [ ] Guarantees **concurrent-safe synchronous reads** without "tearing" (visual inconsistencies caused by an external store updating mid-render).

### 12. `useInsertionEffect` (React 18+)
- [ ] Runs synchronously **before all DOM mutations** and before `useLayoutEffect`.
- [ ] Exclusively intended for CSS-in-JS library authors (e.g. styled-components) to inject `<style>` tags before reading layout.

---

## 6. React 18 & 19 Concurrency & Next-Gen Features

### What is Concurrent React?
- The ability of React to interrupt, pause, resume, or abandon a render.
- Enables the main thread to stay responsive to urgent user interactions (typing, clicking) even while rendering large UI updates in the background.

### 1. Automatic Batching
- Prior to React 18, state updates inside `setTimeout`, `Promise`, or native DOM event listeners were executed in separate re-renders.
- React 18 automatically batches **all** updates into a single render regardless of context:
  ```javascript
  // Only 1 re-render in React 18+:
  fetchData().then(() => {
    setUser(userData);
    setIsLoading(false);
  });
  ```
- To opt out of batching in rare cases: `flushSync(() => setState(...))`.

### 2. Transitions: `useTransition` & `useDeferredValue`
- Splits UI updates into two distinct categories:
  1. **Urgent updates**: Direct user manipulation (typing in input, clicking tabs).
  2. **Transition updates**: Non-urgent transitions (filtering a table, rendering a chart).
```jsx
const [isPending, startTransition] = useTransition();

const handleFilterChange = (e) => {
  setInputValue(e.target.value); // Urgent: immediate input response

  startTransition(() => {
    setSearchQuery(e.target.value); // Non-urgent: can be interrupted by next keystroke
  });
};
```

### 3. React 19 Core Additions
- [ ] **The `use()` Hook**:
  - Can unwrap Promises and Context **conditionally** or inside loops!
  - Unlike standard hooks, `use()` can be called after conditional `if` statements.
  ```jsx
  import { use } from 'react';

  function UserProfile({ userPromise }) {
    const user = use(userPromise); // Suspends until promise resolves!
    return <h1>{user.name}</h1>;
  }
  ```
- [ ] **Actions & `useActionState`**:
  - Manages asynchronous form submissions, handling pending states, errors, and optimistic data automatically.
- [ ] **`useOptimistic`**:
  - Provides instant UI updates during background server mutations, automatically reverting if the server action fails.
- [ ] **`ref` as a Standard Prop**:
  - In React 19, `forwardRef` is deprecated; function components can accept `ref` directly as a prop (`function MyInput({ ref, ...props })`).
- [ ] **React Compiler (Forget)**:
  - An optimizing compiler that automatically injects memoization at build time, eliminating the manual boilerplate of `useMemo`, `useCallback`, and `React.memo`.

---

## 7. Advanced Design Patterns in React

### 1. Compound Components Pattern
Allows multiple components to share implicit state and coordinate with flexible JSX structure:
```jsx
import React, { createContext, useContext, useState } from 'react';

const AccordionContext = createContext();

export function Accordion({ children, defaultOpenId }) {
  const [openId, setOpenId] = useState(defaultOpenId);
  return (
    <AccordionContext.Provider value={{ openId, setOpenId }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

Accordion.Item = function Item({ id, children }) {
  const { openId, setOpenId } = useContext(AccordionContext);
  const isOpen = openId === id;
  return (
    <div className="accordion-item">
      <button onClick={() => setOpenId(isOpen ? null : id)}>Toggle</button>
      {isOpen && <div className="accordion-content">{children}</div>}
    </div>
  );
};
```

### 2. Higher-Order Component (HOC) Pattern
A function that takes a component and returns an enhanced component:
```jsx
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated, loading } = useAuth();
    if (loading) return <div>Loading...</div>;
    if (!isAuthenticated) return <Navigate to="/login" />;
    return <Component {...props} />;
  };
}
```

### 3. Custom Hooks Architecture
Encapsulating reusable business logic and subscriptions:
- `useWindowDimensions()`
- `useDebounce(value, delay)`
- `useLocalStorage(key, initialValue)`
- `usePrevious(value)`
- `useOnClickOutside(ref, handler)`

---

## 8. State Management Ecosystem

| Solution | Paradigm | When to Use | Scalability | Re-render Optimization |
| :--- | :--- | :--- | :--- | :--- |
| **React Context** | Prop distribution | Global themes, auth tokens, localization | Low/Medium | Requires manual splitting or memoization |
| **Redux Toolkit** | Flux (unidirectional) | Large applications, multi-user transactions, complex state logic | Very High | Selector memoization (`useSelector` + shallowEqual) |
| **Zustand** | Store-based (Hooks) | Modern applications wanting lightweight, clean code | High | Atomic selector subscriptions (`useStore(s => s.item)`) |
| **TanStack Query** | Server-state cache | All REST / GraphQL API data fetching | Infinite | Automatic cache invalidation, deduplication |

---

## 9. Performance Optimization & Profiling

### 1. Causes of Unnecessary Re-renders
A component re-renders if:
1. Its state changes (`setState`).
2. Its parent re-renders (regardless of whether props changed!).
3. A Context it subscribes to changes.

### 2. Optimization Arsenal
- [ ] **Lifting Content Up (Component Composition)**: Pass un-related child components as `children` prop. React does not re-render props children when the wrapper's local state changes.
- [ ] **`React.memo`**: Caches component rendered output, re-rendering only when props fail shallow equality.
- [ ] **Stable References**: Use `useCallback` for event handlers and `useMemo` for objects passed to memoized children.
- [ ] **Context Splitting**: Never place frequently changing data and rarely changing data in the same context object.
- [ ] **Virtualization**: Render only elements within the viewport for lists over 100 items (`react-window`, `@tanstack/react-virtual`).

---

## 10. Portals, Error Boundaries & Synthetic Events

### React Portals
- Render children into a DOM node outside the parent component's DOM hierarchy (`document.body` or `#modal-root`):
  ```jsx
  ReactDOM.createPortal(<ModalContent />, document.getElementById('modal-root'));
  ```
- **Crucial Interview Point**: Even though the Portal renders into an arbitrary external DOM node, it behaves like a standard React child in the React tree (context, event bubbling, and lifecycles work as normal).

### Error Boundaries
- Class components implementing `static getDerivedStateFromError()` and/or `componentDidCatch()`.
- Catches JavaScript errors anywhere in their child component tree, logs them, and displays a fallback UI instead of crashing the entire application into a blank white screen.
- *Does NOT catch*: Errors in event handlers, asynchronous code (`setTimeout`, promises), or SSR.

### Synthetic Events & Event Delegation
- React wraps native browser events in a cross-browser wrapper: `SyntheticEvent`.
- **React 17+ Change**: Event listeners are attached to the **root DOM container (`#root`)** instead of the global `document` node, allowing multiple isolated React apps on the same page.

---

## 11. Testing React Components

### Core React Testing Library (RTL) Principles
- "The more your tests resemble the way your software is used, the more confidence they can give you."
- Avoid testing implementation details (internal state, class names, component methods). Test what the user sees and interacts with.

### Query Priority:
1. `getByRole(role, { name })` *(Most accessible, reflects assistive technology)*
2. `getByLabelText` *(Forms)*
3. `getByPlaceholderText`
4. `getByText`
5. `getByTestId` *(Last resort escape hatch)*

---

## 12. Top 20 Real-World React Interview Scenarios & Gotchas

1. **Why is `setState` asynchronous?**
   - Enables batching for performance; guarantees consistency across children by delaying updates until after event handlers execute.
2. **What happens if you pass an object literal directly as a prop?**
   - A new object reference is created every render, breaking `React.memo` on the child.
3. **Why do we need a cleanup function in `useEffect`?**
   - To cancel subscriptions, clear timers, remove native DOM event listeners, and abort pending network requests.
4. **How do you safely fetch data inside `useEffect`?**
   - Use `AbortController` to cancel the request if the component unmounts or dependencies change before the request finishes.
5. **Why can't you call hooks conditionally?**
   - React tracks hooks as a sequential singly-linked list. A condition shifts the order/indices, leading to state corruption.
6. **Can `useMemo` guarantee that a value will never be recalculated?**
   - No, React may free memory by discarding cached values during low-memory conditions; `useMemo` is a performance hint, not a semantic guarantee.
7. **What is "Tearing" in React 18?**
   - When different components render different versions of external state during a concurrent, interruptible render. Solved via `useSyncExternalStore`.
8. **What is Hydration?**
   - The process where client-side JavaScript attaches event listeners and boots state onto server-rendered static HTML.
9. **What causes a Hydration Mismatch error?**
   - Server-rendered HTML differing from client initial render (e.g., using `window.innerWidth`, `Date.now()`, or `Math.random()`).
10. **How do you share state between two sibling components?**
    - Lift state up to their closest common ancestor or use a global store.
11. **What is the difference between `defaultValue` and `value`?**
    - `defaultValue` initializes uncontrolled inputs; `value` locks controlled inputs to React state.
12. **Why does Strict Mode render twice in development?**
    - To help identify accidental side effects in the Render phase (re-running function bodies, reducers, and initializers).
13. **How does React differentiate between mounting and updating in `useEffect`?**
    - Track it with a `useRef`: `const isMounted = useRef(false)`.
14. **What is the difference between shallow and deep comparison in React?**
    - Shallow compares primitive values and object reference equality (`Object.is`); deep comparison recursively checks every nested key.
15. **What is React Fiber's `lanes` model?**
    - A 32-bit integer bitmask representing task priority categories (e.g. Sync, InputContinuous, Default, Transition, Idle).
16. **Why are props immutable in React?**
    - Mutating props directly breaks unidirectional data flow and prevents React from detecting changes via shallow reference comparison.
17. **What is the difference between Client-Side Routing and Server-Side Routing?**
    - Client-side routing intercepts URL changes via HTML5 History API (`pushState`) without reloading the full page from the server.
18. **Can you force a component to reset its entire state?**
    - Yes, change its `key` prop (`<Form key={userId} />`).
19. **How do Server Components (RSC) communicate with Client Components?**
    - Server components pass serializable props (strings, numbers, objects) or pass client components as `children`.
20. **Why should you avoid creating components inside other components?**
    - The inner component is redefined with a new reference on every render, causing React to completely unmount and remount it every time.
