# 🚀 The Ultimate Frontend Interview Preparation & Revision Guide

A comprehensive, production-grade guide designed for frontend software engineers preparing for interviews at top tech companies, startups, and high-growth engineering teams.

> **How to use this guide**:
> - **During long-term preparation**: Go through each module systematically, implement the polyfills from scratch, and practice the system design problems.
> - **24-48 hours before an interview**: Review the [Quick Revision Cheat Sheet & Gotchas](#9-fast-revision-cheat-sheet--common-gotchas) and the [Pre-Interview Checklist](#10-24-hour-pre-interview-checklist).

---

## 📑 Table of Contents

1. [JavaScript Core & Deep Dive](#1-javascript-core--deep-dive)
2. [TypeScript Essentials](#2-typescript-essentials)
3. [HTML5, Web APIs & Browser Internals](#3-html5-web-apis--browser-internals)
4. [CSS & Modern Layouts](#4-css--modern-layouts)
5. [React & State Architecture](#5-react--state-architecture)
6. [Web Performance & Core Web Vitals](#6-web-performance--core-web-vitals)
7. [Frontend System Design & Architecture](#7-frontend-system-design--architecture)
8. [Machine Coding & Polyfills (Code-From-Scratch)](#8-machine-coding--polyfills-must-know)
9. [Web Accessibility (a11y) & Testing](#9-web-accessibility-a11y--testing)
10. [Fast Revision Cheat Sheet & Common Gotchas](#10-fast-revision-cheat-sheet--common-gotchas)
11. [24-Hour Pre-Interview Checklist](#11-24-hour-pre-interview-checklist)

---

## 1. JavaScript Core & Deep Dive

### Execution Context & Call Stack
- [ ] **Global Execution Context (GEC)** vs **Function Execution Context (FEC)**.
- [ ] **Two phases of execution**:
  1. *Creation Phase (Memory Allocation)*: Variables initialized to `undefined` (`var`), functions stored in memory, `window`/`global` and `this` established.
  2. *Execution Phase*: Code evaluated and executed line-by-line.
- [ ] **Call Stack**: LIFO structure tracking running contexts. Stack overflow happens when maximum stack depth is exceeded (e.g., unbounded recursion).

### Scopes, Hoisting & Closures
- [ ] **Scope Types**: Global scope, Function scope, Block scope (`let`, `const`).
- [ ] **Lexical Scope**: Inner functions have access to variables declared in outer enclosing scopes determined at parse/definition time.
- [ ] **Hoisting & Temporal Dead Zone (TDZ)**:
  - `var` declarations are hoisted and initialized to `undefined`.
  - `let` & `const` are hoisted into the TDZ: accessing them before declaration throws `ReferenceError`.
  - Function declarations are hoisted completely with their definitions. Function expressions assigned to variables behave like their variable declarations.
- [ ] **Closures**:
  - Definition: A function bundled together with references to its lexical environment.
  - Practical Use Cases: Data privacy/encapsulation (module pattern), memoization, function currying, maintaining state in asynchronous callbacks.
  - Memory leak risk: Closures keeping references to large DOM nodes or obsolete objects in outer scopes.

```javascript
// Classic Closure Gotcha: var vs let in loops
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Prints: 3, 3, 3 (shared function scope)
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100); // Prints: 0, 1, 2 (per-iteration block scope)
}
```

### Event Loop & Asynchronous Architecture
- [ ] **Call Stack**: Executes synchronous code.
- [ ] **Web APIs / Node APIs**: Offloads timers, network requests, DOM events, file I/O.
- [ ] **Microtask Queue** (Highest Priority):
  - `Promise.then()` / `catch()` / `finally()`
  - `queueMicrotask()`
  - `MutationObserver`
  - `process.nextTick()` (Node.js specific, runs before other microtasks)
- [ ] **Macrotask / Task Queue** (Lower Priority):
  - `setTimeout()`, `setInterval()`
  - `setImmediate()` (Node.js)
  - DOM events, I/O operations
  - `requestAnimationFrame()` (runs right before the browser repaints)
- [ ] **Execution Rule**: After every synchronous execution or task completes, the engine drains the **entire Microtask Queue** before executing the next Macrotask or performing UI rendering/repainting.

```
       ┌────────────────────────┐
       │       Call Stack       │
       └───────────┬────────────┘
                   │ Empty?
       ┌───────────▼────────────┐
       │   Microtask Queue      │ ◄── [Promises, queueMicrotask] (DRAIN ALL)
       └───────────┬────────────┘
                   │ Empty?
       ┌───────────▼────────────┐
       │   Browser Repaint /    │ ◄── [requestAnimationFrame, Render]
       │   Rendering Pipeline   │
       └───────────┬────────────┘
                   │
       ┌───────────▼────────────┐
       │    Macrotask Queue     │ ◄── [setTimeout, setInterval, I/O] (RUN ONE)
       └────────────────────────┘
```

### `this` Keyword & Binding Rules
| Binding Precedence | Mechanism | Example |
| :--- | :--- | :--- |
| 1. `new` Binding | Constructor call | `const obj = new Car();` (`this` is new object) |
| 2. Explicit Binding | `.call()`, `.apply()`, `.bind()` | `fn.call(context, arg1)` / `fn.apply(context, [args])` |
| 3. Implicit Binding | Method on object | `user.getName()` (`this` points to `user`) |
| 4. Default Binding | Standalone call | `fn()` (`window` in non-strict, `undefined` in strict) |
| Special: Arrow Functions | Lexical scope | Inherits `this` from the enclosing lexical scope; cannot be overridden by `.bind()` |

### Prototypes & Prototypal Inheritance
- [ ] Difference between `.prototype` (property on constructor functions) and `__proto__` / `Object.getPrototypeOf()` (internal link on instances).
- [ ] Prototype chain traversal: if property not found on instance, JS walks up prototype chain until `null` (`Object.prototype.__proto__ === null`).
- [ ] ES6 `class` is syntactic sugar over prototype-based inheritance.
- [ ] Methods declared on `class` live on `Class.prototype`, fields initialized with `=` live on each instance.

### Objects, Iteration & Memory
- [ ] **Primitive Types**: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
- [ ] **Reference Types**: `Object`, `Array`, `Function`, `Map`, `Set`, `WeakMap`, `WeakSet`, `Date`, `RegExp`.
- [ ] **Garbage Collection**: Mark-and-sweep algorithm. Roots include global variables and active stack frames.
- [ ] **Memory Leaks in JS**:
  1. Accidental global variables (`window.foo = ...`).
  2. Uncleared `setInterval` or `setTimeout`.
  3. Detached DOM nodes retained in JS variables.
  4. Obsolete event listeners not removed on component unmount.
  5. Closures retaining unneeded variables.

---

## 2. TypeScript Essentials

### Core Types & System
- [ ] `unknown` (type-safe counterpart of `any`, requires narrowing before use) vs `any` (disables type checking) vs `never` (represents values that never occur, e.g., functions throwing or infinite loops).
- [ ] `type` vs `interface`:
  - `interface` supports declaration merging; ideal for public API surface and OO modeling.
  - `type` supports unions, intersections, mapped types, tuples, primitives.
- [ ] **Type Narrowing**: `typeof`, `instanceof`, `in` operator, user-defined type guards (`val is MyType`).
- [ ] **Discriminated Unions (Tagged Unions)**:
  ```typescript
  type Success = { status: 'success'; data: string[] };
  type ErrorState = { status: 'error'; message: string };
  type ResponseState = Success | ErrorState;

  function handle(res: ResponseState) {
    if (res.status === 'success') {
      console.log(res.data); // Narrowed to Success
    } else {
      console.error(res.message); // Narrowed to ErrorState
    }
  }
  ```

### Advanced Generics & Built-in Utility Types
- [ ] `Partial<T>`, `Required<T>`, `Readonly<T>`.
- [ ] `Pick<T, K>`, `Omit<T, K>`, `Record<K, T>`.
- [ ] `Exclude<T, U>`, `Extract<T, U>`, `NonNullable<T>`.
- [ ] `ReturnType<T>`, `Parameters<T>`, `Awaited<T>`.
- [ ] `keyof`, `typeof`, indexed access types (`T[K]`), mapped types (`[P in K]: T`).

---

## 3. HTML5, Web APIs & Browser Internals

### Critical Rendering Path (CRP)
```
1. HTML Byte Stream ──> Tokenization ──> Nodes ──> DOM Tree
2. CSS Byte Stream  ──> Tokenization ──> Nodes ──> CSSOM Tree
3. DOM + CSSOM      ──> Render Tree (excludes display: none, head, script)
4. Layout (Reflow)  ──> Computes geometric position and dimensions (box model)
5. Paint (Repaint)  ──> Fills pixels (colors, borders, text, shadows)
6. Compositing      ──> Layers sent to GPU for rasterization (transform, opacity)
```

- [ ] **Reflow vs Repaint**:
  - *Reflow triggers*: changing layout properties (`width`, `height`, `margin`, `padding`, `top`, `fontSize`, adding/removing DOM elements, reading `offsetWidth`/`scrollTop` forcing layout flush).
  - *Repaint triggers*: visual properties that don't affect layout (`color`, `background-color`, `visibility`).
  - *GPU composited properties* (No reflow, no repaint): `transform`, `opacity`.

### Script Loading & Resource Hints
- [ ] `<script src="...">`: Blocks HTML parsing until fetched and executed.
- [ ] `<script async src="...">`: Fetched asynchronously in background; executed **immediately** once fetched, interrupting parser. Order not guaranteed.
- [ ] `<script defer src="...">`: Fetched asynchronously in background; executed **in order** only after HTML parsing completes, before `DOMContentLoaded`.
- [ ] `<script type="module">`: Defer-like behavior by default, runs in strict mode, supports top-level `import`/`export`.
- [ ] **Resource Hints**:
  - `dns-prefetch`: Resolves DNS early (`<link rel="dns-prefetch" href="...">`).
  - `preconnect`: DNS lookup + TCP handshake + TLS negotiation.
  - `preload`: Mandatory high-priority fetch for resources needed on current page (critical fonts, hero images).
  - `prefetch`: Low-priority fetch for resources likely needed on subsequent navigations.

### DOM Events & Event Delegation
- [ ] **Event Phases**:
  1. *Capturing Phase* (window down to target).
  2. *Target Phase*.
  3. *Bubbling Phase* (target up to window - default for `addEventListener(..., false)`).
- [ ] `e.target` (element that triggered event) vs `e.currentTarget` (element to which the listener is attached).
- [ ] `e.stopPropagation()` (stops bubbling/capturing) vs `e.stopImmediatePropagation()` (stops other handlers on same element) vs `e.preventDefault()` (cancels default browser behavior).
- [ ] **Event Delegation**: Attaching a single listener to a common parent instead of hundreds of child nodes. Leverages event bubbling; reduces memory footprint and supports dynamically added elements.

### Browser Storage & State Persistence
| Storage | Capacity | Lifetime | Accessible By | Transmission with HTTP | Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LocalStorage** | ~5-10MB | Persistent until cleared | Same origin | No | User theme, persistent settings |
| **SessionStorage** | ~5MB | Tab/session lifetime | Same origin & tab | No | Single-session wizard, temporary form |
| **Cookies** | ~4KB | Expiration date | Origin/path | Yes (every HTTP request) | Auth session tokens (`HttpOnly`, `Secure`) |
| **IndexedDB** | >50MB (quota) | Persistent | Same origin | No | Large structured offline datasets |

### Web Security Essentials
- [ ] **Cross-Origin Resource Sharing (CORS)**:
  - Same-Origin Policy (Protocol + Domain + Port).
  - Simple requests vs Preflight request (`OPTIONS` checking `Access-Control-Allow-Origin`, `Methods`, `Headers`).
- [ ] **Cross-Site Scripting (XSS)**:
  - Types: Stored (DB), Reflected (URL query), DOM-based (client JS using `innerHTML` or `eval`).
  - Mitigations: Context-aware HTML escaping, DOMPurify, Content Security Policy (CSP), `HttpOnly` cookies.
- [ ] **Cross-Site Request Forgery (CSRF)**:
  - Unauthorized commands submitted on behalf of authenticated user.
  - Mitigations: `SameSite=Strict/Lax` cookie attribute, Anti-CSRF tokens, Custom headers (`X-Requested-With`).
- [ ] **Clickjacking**: Mitigated via `X-Frame-Options: DENY / SAMEORIGIN` and CSP `frame-ancestors`.

---

## 4. CSS & Modern Layouts

### The Box Model
- [ ] `box-sizing: content-box` (default: width = content only; padding & border add up).
- [ ] `box-sizing: border-box` (preferred: width includes content + padding + border).
- [ ] **Margin Collapsing**: Adjacent vertical margins of block-level elements collapse to the largest single margin. Does not collapse in Flexbox or Grid children.

### Flexbox vs CSS Grid
- [ ] **Flexbox (1D - row OR column)**:
  - Container: `display: flex`, `flex-direction`, `justify-content` (main axis), `align-items` (cross axis), `flex-wrap`.
  - Item: `flex-grow`, `flex-shrink`, `flex-basis` (shorthand `flex: 1 1 0%`), `align-self`.
- [ ] **CSS Grid (2D - rows AND columns)**:
  - `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))`.
  - `gap`, `grid-template-areas`, `subgrid`.

### Stacking Context & `z-index`
- [ ] A stacking context is formed by:
  - Root element `<html>`.
  - Positioned element (`relative`/`absolute`) with `z-index` not `auto`.
  - `position: fixed` or `sticky`.
  - Elements with `opacity < 1`, `transform`, `filter`, `perspective`, `clip-path`.
  - `isolation: isolate`.
- [ ] **Golden rule**: An element with higher `z-index` inside a lower stacking context will NEVER render above an element in a higher stacking context.

### Modern CSS Selectors & Features
- [ ] **Specificity scoring**: `!important` > Inline (`1000`) > ID (`100`) > Class / Attribute / Pseudo-class (`10`) > Element / Pseudo-element (`1`).
- [ ] `:is()` (takes highest specificity inside) vs `:where()` (zero specificity).
- [ ] `:has()` parent selector (`card:has(img)`).
- [ ] Container queries (`@container (min-width: 400px)`).
- [ ] Fluid typography with `clamp(min, preferred, max)` (e.g., `font-size: clamp(1rem, 2.5vw, 2rem)`).

---

## 5. React & State Architecture

### React Internals & Fiber Architecture
- [ ] **Virtual DOM (VDOM)**: In-memory lightweight JS tree representing the UI.
- [ ] **Diffing Algorithm Assumptions**:
  1. Elements of different types produce different trees (unmount old, mount new).
  2. The developer can hint at stability across renders with the `key` prop.
  3. Never use array index as `key` for re-orderable, filterable, or dynamic lists (causes component state mismatch and DOM reuse bugs).
- [ ] **Fiber Engine**:
  - Incremental rendering: Ability to split work into chunks and pause/resume execution across browser animation frames.
  - Priority-based scheduling (User inputs > Data fetching/background animations).
  - Two Phases:
    1. *Render Phase* (reconciliation, pure, asynchronous, can be paused/aborted).
    2. *Commit Phase* (DOM mutation, synchronous, runs layout effects and passive effects).

### Deep Dive into React Hooks
- [ ] **`useState`**: Stores component local state; triggers re-render when setter is called with a new reference (`Object.is`).
- [ ] **`useEffect` vs `useLayoutEffect`**:
  - `useEffect`: Runs **asynchronously after** the browser has painted the screen. Used for data fetching, subscriptions, timer setup.
  - `useLayoutEffect`: Runs **synchronously after** DOM mutations, but **before** the browser paints. Used to measure DOM dimensions or prevent visual flickers.
- [ ] **`useMemo` & `useCallback`**:
  - `useMemo`: Caches computed value across renders (`useMemo(() => computeHeavy(a), [a])`).
  - `useCallback`: Caches function instance across renders (`useCallback(fn, deps) === useMemo(() => fn, deps)`).
  - Use when: Passing callbacks to optimized child components wrapped in `React.memo`, or preventing re-computation of heavy algorithms.
- [ ] **`useRef`**: Persists mutable value across renders without causing a re-render when mutated. Accesses DOM nodes or stores interval IDs.
- [ ] **Rules of Hooks**: Only call at top level (not inside conditions or loops); only call from React functions. Enabled by React storing hooks as a sequential linked list on each Fiber.

### React 18 & 19 Concurrency
- [ ] **Automatic Batching**: Batches multiple state updates into a single re-render even inside Promises, `setTimeout`, and native event listeners.
- [ ] **Transitions**:
  - `useTransition`: Splits updates into urgent (typing, clicking) and non-urgent (filtering large lists, page transitions).
  - `useDeferredValue`: Defers updating a non-critical portion of UI based on input changes.
- [ ] **Suspense for Data Fetching & Streaming SSR**: Enables progressive rendering from the server with fallback states.
- [ ] **React Server Components (RSC)**: Run exclusively on server; zero client bundle size for imported dependencies; direct DB/filesystem access.

### State Management Paradigms
- [ ] **Local State**: `useState`, `useReducer`.
- [ ] **Context API**: Best for low-frequency global updates (theme, user locale, auth token). Pitfall: any context value change triggers re-renders on all consuming components unless memoized or split.
- [ ] **Flux / Redux (Redux Toolkit)**: Unidirectional data flow (Action -> Reducer -> Store -> View). Immutability with Immer. Middleware (Thunks) for async side effects.
- [ ] **Atomic State (Jotai, Recoil)**: Granular atoms; fine-grained re-renders.
- [ ] **Server State Management (TanStack Query / SWR)**:
  - Cache invalidation, deduplication, background refetching, pagination, infinite loading, optimistic mutations.

---

## 6. Web Performance & Core Web Vitals

### Core Web Vitals (CWV)
| Metric | Full Name | Good Target | What It Measures | Optimization Strategies |
| :--- | :--- | :--- | :--- | :--- |
| **LCP** | Largest Contentful Paint | $\le 2.5\text{s}$ | Loading speed of main content | Use modern formats (AVIF/WebP), CDN, `fetchpriority="high"` on hero img, preload critical font/image, server-side caching |
| **INP** | Interaction to Next Paint | $\le 200\text{ms}$ | Responsiveness to user interactions | Yield long tasks to main thread (`scheduler.yield()`, `setTimeout(0)`), avoid layout thrashing, debounce fast typing events |
| **CLS** | Cumulative Layout Shift | $\le 0.1$ | Visual stability during page load | Set explicit `width`/`height` or `aspect-ratio` on images/videos, reserve skeleton space for ads/dynamic widgets, `font-display: swap` |

### Network & Asset Optimization
- [ ] **HTTP Evolution**:
  - HTTP/1.1: Persistent connections, head-of-line blocking at application layer.
  - HTTP/2: Binary protocol, single TCP connection with multiplexed streams, server push, header compression (HPACK).
  - HTTP/3: Runs on UDP (QUIC protocol), eliminates TCP head-of-line blocking on packet drop, zero-RTT connection resumption.
- [ ] **Caching Headers**:
  - `Cache-Control: max-age=31536000, immutable` (ideal for hashed static assets).
  - `Cache-Control: no-cache` (must validate with server via `ETag` before serving).
  - `Cache-Control: no-store` (never store in any cache; used for sensitive data).
  - `stale-while-revalidate=60` (serve stale content instantly while revalidating in background).
- [ ] **Code Splitting & Bundle Optimization**:
  - Route-based code splitting (`React.lazy()` + dynamic `import()`).
  - Tree shaking (requires ES module syntax `import`/`export` and `sideEffects: false` in `package.json`).
  - Analyzing bundle size using `webpack-bundle-analyzer` or Vite bundle visualizer.

### Rendering Patterns Matrix
| Pattern | Where it Renders | TTFB | FCP / LCP | SEO | Complexity |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CSR** (Client-Side) | Browser | Fast | Slow | Poor / Average | Low |
| **SSR** (Server-Side) | Node/Edge Server | Slower | Fast | Excellent | Medium / High |
| **SSG** (Static Site Gen) | Build time | Fastest | Fastest | Excellent | Low |
| **ISR** (Incremental) | Server on interval | Fastest | Fastest | Excellent | Medium |
| **Streaming SSR** | Server chunk-by-chunk | Fast | Very Fast | Excellent | High |

---

## 7. Frontend System Design & Architecture

### The Standard 4-Step System Design Framework
1. **Requirements Gathering (5-8 mins)**:
   - Functional requirements (e.g., infinite scroll, real-time sync, offline mode).
   - Non-functional requirements (device/network constraints, 60fps scrolling, INP $\le 100\text{ms}$, accessibility).
2. **High-Level Component Hierarchy & Data Model (10-15 mins)**:
   - Identify UI components and boundary divisions.
   - Design client-side state schema (normalized data structure `byId` and `allIds`).
   - Define API contracts (REST / GraphQL / WebSocket payloads).
3. **Deep Dive Implementation (15-20 mins)**:
   - Virtualization / Windowing algorithm for infinite lists.
   - Caching, pagination strategies (offset vs cursor-based).
   - Optimistic UI updates with rollback on error.
   - Network resiliency (retry with exponential backoff, request deduplication).
4. **Edge Cases & Optimization (5-10 mins)**:
   - Offline handling (Service Worker, IndexedDB sync).
   - Accessibility & keyboard navigation.
   - Memory management (recycling DOM nodes, garbage collection safety).

### Essential System Design Case Studies
- [ ] **Autocomplete / Search Typeahead**:
  - Debouncing keystrokes, Trie data structure / cache in memory.
  - Race condition prevention using `AbortController` (discard stale out-of-order network responses).
  - ARIA combobox pattern (`aria-autocomplete`, `aria-activedescendant`).
- [ ] **Infinite Scroll / News Feed**:
  - Virtualization (`react-window`, `react-virtualized`): Render only items visible in viewport + buffer.
  - Cursor-based pagination (`cursor=timestamp_id`) to avoid duplicates when new items are posted.
  - Image lazy loading with `IntersectionObserver`.
- [ ] **Real-Time Collaborative Document / Chat**:
  - WebSockets vs Server-Sent Events (SSE) vs Polling.
  - Conflict resolution: Operational Transformation (OT) vs Conflict-free Replicated Data Types (CRDT).
  - Optimistic message states: *sending* $\to$ *sent* $\to$ *delivered* $\to$ *failed (retry)*.
- [ ] **Drag & Drop Kanban Board (Trello-like)**:
  - HTML5 Drag and Drop API vs Pointer Events.
  - Immediate state reordering in React local state with backend synchronization.

---

## 8. Machine Coding & Polyfills (Must-Know)

### 1. `debounce` (with `immediate` option)
```javascript
function debounce(fn, delay, immediate = false) {
  let timerId = null;

  function debounced(...args) {
    const context = this;
    const callNow = immediate && !timerId;

    clearTimeout(timerId);

    timerId = setTimeout(() => {
      timerId = null;
      if (!immediate) fn.apply(context, args);
    }, delay);

    if (callNow) fn.apply(context, args);
  }

  debounced.cancel = () => {
    clearTimeout(timerId);
    timerId = null;
  };

  return debounced;
}
```

### 2. `throttle` (leading & trailing options)
```javascript
function throttle(fn, limit) {
  let inThrottle = false;
  let lastArgs = null;
  let lastContext = null;

  return function throttled(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
        if (lastArgs) {
          throttled.apply(lastContext, lastArgs);
          lastArgs = null;
          lastContext = null;
        }
      }, limit);
    } else {
      lastArgs = args;
      lastContext = this;
    }
  };
}
```

### 3. `Promise.all` Polyfill
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }

    const results = [];
    let completedCount = 0;
    const total = promises.length;

    if (total === 0) return resolve(results);

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((value) => {
          results[index] = value;
          completedCount += 1;
          if (completedCount === total) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

### 4. `Promise.allSettled` Polyfill
```javascript
function promiseAllSettled(promises) {
  return Promise.all(
    promises.map((p) =>
      Promise.resolve(p)
        .then((value) => ({ status: 'fulfilled', value }))
        .catch((reason) => ({ status: 'rejected', reason }))
    )
  );
}
```

### 5. Deep Clone (Handling Circular References & Types)
```javascript
function deepClone(obj, hash = new WeakMap()) {
  if (Object(obj) !== obj) return obj; // Primitive
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (hash.has(obj)) return hash.get(obj); // Handle cyclic graph

  const result = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, result);

  for (const key of Reflect.ownKeys(obj)) {
    result[key] = deepClone(obj[key], hash);
  }

  return result;
}
```

### 6. Event Emitter (Pub-Sub)
```javascript
class EventEmitter {
  constructor() {
    this.events = new Map();
  }

  on(eventName, listener) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    this.events.get(eventName).push(listener);
    return () => this.off(eventName, listener);
  }

  emit(eventName, ...args) {
    const listeners = this.events.get(eventName);
    if (listeners) {
      listeners.slice().forEach((fn) => fn(...args));
    }
  }

  off(eventName, listener) {
    const listeners = this.events.get(eventName);
    if (!listeners) return;
    this.events.set(
      eventName,
      listeners.filter((fn) => fn !== listener)
    );
  }

  once(eventName, listener) {
    const wrapper = (...args) => {
      this.off(eventName, wrapper);
      listener(...args);
    };
    return this.on(eventName, wrapper);
  }
}
```

### 7. Function Currying
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...nextArgs) {
      return curried.apply(this, args.concat(nextArgs));
    };
  };
}
```

---

## 9. Web Accessibility (a11y) & Testing

### Accessibility Principles (WCAG)
- [ ] **POUR**:
  - *Perceivable*: Alt text for images, captions for video, minimum 4.5:1 text contrast.
  - *Operable*: Keyboard navigable throughout; no keyboard traps; skip navigation link.
  - *Understandable*: Predictable behavior, clear validation errors on forms.
  - *Robust*: Works across modern browsers and assistive technologies (screen readers like NVDA/VoiceOver).
- [ ] **ARIA Landmarks & Roles**:
  - Use native HTML elements first (`<main>`, `<nav>`, `<header>`, `<footer>`, `<button>`, `<dialog>`).
  - Don't use `<div onClick="...">` when `<button>` exists.
  - `aria-live="polite"` (announces after current speech) vs `aria-live="assertive"` (interrupts user immediately for alerts).
  - Focus trap inside Modals: When a modal opens, focus moves inside and `Tab` cycles within modal; `Escape` closes it and restores focus to triggering element.

### Testing Strategies & Tools
- [ ] **Testing Pyramid**:
  - **Unit Tests** (Vitest / Jest): Pure utilities, reducers, custom hooks (`@testing-library/react-hooks`).
  - **Component Tests** (React Testing Library): Test user behavior, not implementation details. Query priority: `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > `getByTestId`.
  - **End-to-End (E2E)** (Playwright / Cypress): Critical user funnels (Sign-in, Cart checkout, Payment flow).
  - **Visual Regression**: Chromatic, Percy.

---

## 10. Fast Revision Cheat Sheet & Common Gotchas

### JavaScript Quirks & Blitz Questions
```javascript
// 1. Primitive Coercion
console.log([] + []);          // "" (both coerced to empty strings)
console.log([] + {});          // "[object Object]"
console.log({} + []);          // 0 (in raw block console) or "[object Object]"
console.log(typeof null);       // "object" (legacy JS bug)
console.log(null instanceof Object); // false

// 2. NaN Checks
console.log(NaN === NaN);       // false
console.log(Number.isNaN('x')); // false (safe: only true for actual NaN value)
console.log(isNaN('x'));        // true (unsafe: coerces 'x' to number first)

// 3. Floating Point Precision
console.log(0.1 + 0.2 === 0.3); // false (0.30000000000000004)
// Correct way:
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON); // true

// 4. Object Keys
const a = {}, b = { key: 'b' }, c = { key: 'c' };
a[b] = 123; // a["[object Object]"] = 123
a[c] = 456; // a["[object Object]"] = 456
console.log(a[b]); // 456

// 5. Array Sort Default
console.log([10, 5, 20, 2].sort()); // [10, 2, 20, 5] (lexicographical string sort!)
// Correct numerical sort:
console.log([10, 5, 20, 2].sort((x, y) => x - y)); // [2, 5, 10, 20]
```

### Event Loop Output Puzzle
```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => console.log('3'));
}, 0);

new Promise((resolve) => {
  console.log('4'); // Synchronous executor!
  resolve();
}).then(() => {
  console.log('5');
});

console.log('6');

// Output order: 1, 4, 6, 5, 2, 3
```

---

## 11. 24-Hour Pre-Interview Checklist

- [ ] **Polyfills Flashcard Practice**: Can you code `debounce`, `throttle`, and `Promise.all` in 5 minutes without syntax errors?
- [ ] **React State Update Behavior**: Know when batching happens and how to avoid stale state using functional updates `setVal(prev => prev + 1)`.
- [ ] **CSS Centering**: Can effortlessly center with `display: grid; place-items: center` or Flexbox.
- [ ] **Core Web Vitals Metric Names**: LCP, INP, CLS — know their target thresholds and how to diagnose them.
- [ ] **Event Delegation**: Know how to explain `target` vs `currentTarget` and event capturing vs bubbling.
- [ ] **Machine Coding Readiness**:
  - Keep your development environment ready (Node, Vite/Next template, TypeScript configured).
  - Practice setting up a clean component structure with accessible HTML elements first before adding styles.
- [ ] **Behavioral (STAR Method)**:
  - Prepare 2-3 stories: A challenging bug / production issue you solved, an architectural decision you defended, and how you handled cross-functional conflict with product or design.

---

*Good luck with your interviews! Star and bookmark this repository for quick reference.*
