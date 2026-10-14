# 🚀 Frontend Interview Preparation Master Hub

Welcome to the ultimate frontend interview preparation and revision repository. This repository is organized into dedicated, comprehensive topic guides so you can systematically prepare and quickly revise every major area of frontend engineering.

---

## 🗺️ Topic Roadmaps & Specialized Guides

Click into any guide below for an exhaustive deep-dive, code snippets, internal mechanisms, and interview practice questions:

| Topic Guide | Description | Key Areas Covered |
| :--- | :--- | :--- |
| [**⚡ JavaScript Deep Dive**](javascript.md) | The language core, execution engine, and polyfills | Execution Context, Call Stack, Scopes, Closures, Hoisting, `this`, Prototypes, Event Loop, Promises, ES6+, Polyfills |
| [**⚛️ React.js Deep Dive**](react.md) | Modern React architecture and state management | Virtual DOM, Fiber Engine, Reconciliation, Hooks in-depth, React 18/19 Concurrency, Server Components (RSC), Performance |
| [**🔷 TypeScript Deep Dive**](typescript.md) | Type systems, advanced generics & typing React | `unknown`/`any`/`never`, Discriminated Unions, Type Narrowing, Generics, Utility Types, Typing React Props & Events |
| [**🌐 HTML5, Browser Internals & Web APIs**](html-browser.md) | Browser architecture and security | Critical Rendering Path (CRP), Reflow vs Repaint, Script loading (`async`/`defer`), Event Delegation, Storage, CORS, XSS, CSRF |
| [**🎨 CSS & Modern Layouts**](css.md) | Layouts, styling architectures and animation | Box Model, Stacking Context (`z-index`), Flexbox, CSS Grid, Fluid Typography (`clamp`), Container Queries, `:has()` |
| [**⚡ Web Performance & Core Web Vitals**](web-performance.md) | Speed, network, and metric optimization | LCP, INP, CLS, HTTP/2 vs HTTP/3, Cache-Control, Code Splitting, Tree Shaking, Rendering Patterns (CSR/SSR/SSG/ISR) |
| [**📐 Frontend System Design**](frontend-system-design.md) | Large-scale frontend application architecture | 4-step framework, Normalized State, Autocomplete, Infinite Scroll Virtualization, Real-time Chat/WebSockets, Offline Sync |
| [**💻 Machine Coding & UI Components**](machine-coding.md) | Practical 45-60 min coding round widgets | Star Rating, Modal with Focus Trap, Autocomplete, Toast system, Nested Comments, Infinite Scroll, Carousel |

---

## ⚡ 24-Hour Pre-Interview Rapid Blitz Checklist

Before heading into your interview, make sure you can answer and implement these top checkpoints:

### 1. JavaScript Must-Knows
- [ ] Can you implement `debounce` and `throttle` from memory in under 3 minutes? ([Review in javascript.md](javascript.md#10-core-polyfills--essential-utilities))
- [ ] Can you explain the exact difference between Microtasks (`Promise.then`, `queueMicrotask`) and Macrotasks (`setTimeout`, I/O)? ([Review in javascript.md](javascript.md#7-asynchronous-javascript--the-event-loop))
- [ ] Can you implement `Promise.all` handling rejection and empty arrays? ([Review in javascript.md](javascript.md#10-core-polyfills--essential-utilities))
- [ ] Can you explain the 4 rules of `this` binding and how arrow functions differ? ([Review in javascript.md](javascript.md#5-the-this-keyword--explicit-binding))

### 2. React Must-Knows
- [ ] What is the difference between the **Render Phase** and the **Commit Phase** in React Fiber? ([Review in react.md](react.md#3-component-lifecycle--render-phases))
- [ ] When would you use `useLayoutEffect` instead of `useEffect`? ([Review in react.md](react.md#4-hooks-deep-dive--internals))
- [ ] What are the dangers of stale closures in hooks, and how do you resolve them? ([Review in react.md](react.md#10-common-react-pitfalls--interview-questions))
- [ ] How does Automatic Batching work in React 18+? ([Review in react.md](react.md#5-react-18--19-concurrency-features))

### 3. Browser & Performance Must-Knows
- [ ] Walk through the **Critical Rendering Path (CRP)** step by step. ([Review in html-browser.md](html-browser.md#1-the-critical-rendering-path-crp))
- [ ] What causes Layout Thrashing (forced synchronous layout), and how do you prevent it? ([Review in html-browser.md](html-browser.md#2-reflow-layout-vs-repaint-paint))
- [ ] Name the 3 Core Web Vitals (**LCP**, **INP**, **CLS**), their targets, and how to fix them. ([Review in web-performance.md](web-performance.md#1-core-web-vitals-lcp-inp-cls))
- [ ] What is the difference between `Cache-Control: no-cache` and `no-store`? ([Review in web-performance.md](web-performance.md#3-http-caching--headers))

### 4. CSS & Styling Must-Knows
- [ ] What triggers a new **Stacking Context** and why does high `z-index` fail inside a lower stacking context? ([Review in css.md](css.md#3-positioning--stacking-context-z-index))
- [ ] Flexbox vs Grid: when do you choose one over the other? ([Review in css.md](css.md#4-flexbox-layout-system))
- [ ] How do you achieve 60fps animations without triggering reflows? ([Review in css.md](css.md#8-animations-transitions--60fps-rendering))

### 5. Frontend System Design Must-Knows
- [ ] Can you articulate the 4-step framework (Requirements $\to$ Architecture/Contracts $\to$ Deep Dives $\to$ Edge Cases)? ([Review in frontend-system-design.md](frontend-system-design.md#1-the-4-step-system-design-interview-framework))
- [ ] How do you handle race conditions in search typeaheads (`AbortController`)? ([Review in frontend-system-design.md](frontend-system-design.md#4-case-study-1-autocomplete--typeahead-search))
- [ ] How does DOM virtualization work for 100,000 feed items? ([Review in frontend-system-design.md](frontend-system-design.md#5-case-study-2-infinite-scroll--virtualized-feed))

---

## 🎯 Behavioral & Engineering Leadership (STAR Method)

Top frontend interviews dedicate 30-45 minutes to behavioral questions. Prepare 2-3 stories using the **STAR method** (Situation, Task, Action, Result):

1. **Resolving a Complex Production Bug**:
   - *Prompt*: "Tell me about a challenging bug you diagnosed and fixed in production."
   - *Key Frontend Points*: Memory leak tracing via Chrome DevTools Heap Snapshots, bundle size regression, cross-browser rendering anomaly.
2. **Technical Conflict / Architectural Disagreement**:
   - *Prompt*: "Tell me about a time you disagreed with a product manager or tech lead on technical approach."
   - *Key Frontend Points*: Balancing feature velocity vs technical debt, advocating for accessibility/performance budgets, providing data-backed alternatives.
3. **Performance Optimization Win**:
   - *Prompt*: "Tell me about a time you significantly improved application performance."
   - *Key Frontend Points*: Measured CWV before/after, identified LCP/INP bottlenecks, split code bundles, reduced TTFB or bundle size by X%.

---

## 📈 Suggested Study Schedule (4-Week Plan)

- **Week 1: Foundations** $\to$ [JavaScript](javascript.md) & [TypeScript](typescript.md)
- **Week 2: Rendering & Platform** $\to$ [HTML & Browser Internals](html-browser.md) & [CSS](css.md)
- **Week 3: Framework & Speed** $\to$ [React.js](react.md) & [Web Performance](web-performance.md)
- **Week 4: Synthesis & Hands-On** $\to$ [Frontend System Design](frontend-system-design.md) & [Machine Coding Practice](machine-coding.md)

---

*Star and clone this repository to track your preparation checklist!*
