# ⚡ JavaScript Comprehensive Interview Guide

A deep-dive, exhaustive guide covering all core concepts, edge cases, internal mechanisms, polyfills, and interview questions for JavaScript.

---

## 📑 Table of Contents
1. [JavaScript Engine & Execution Context](#1-javascript-engine--execution-context)
2. [Memory Management & Garbage Collection](#2-memory-management--garbage-collection)
3. [Scopes, Scope Chain & Closures](#3-scopes-scope-chain--closures)
4. [Hoisting & The Temporal Dead Zone (TDZ)](#4-hoisting--the-temporal-dead-zone-tdz)
5. [The `this` Keyword & Explicit Binding](#5-the-this-keyword--explicit-binding)
6. [Prototypes & Prototypal Inheritance](#6-prototypes--prototypal-inheritance)
7. [Asynchronous JavaScript & The Event Loop](#7-asynchronous-javascript--the-event-loop)
8. [Promises, Async/Await & Microtask Scheduling](#8-promises-asyncawait--microtask-scheduling)
9. [ES6+ Modern Features & Advanced Metaprogramming](#9-es6-modern-features--advanced-metaprogramming)
10. [Core Polyfills & Essential Utilities](#10-core-polyfills--essential-utilities)
11. [Tricky JS Output Questions & Gotchas](#11-tricky-js-output-questions--gotchas)

---

## 1. JavaScript Engine & Execution Context

### JS Engine Architecture (V8, JavaScriptCore, SpiderMonkey)
- [ ] **Parser**: Converts source code into an Abstract Syntax Tree (AST).
- [ ] **Interpreter (Ignition in V8)**: Converts AST into bytecode for rapid initial execution.
- [ ] **JIT Compiler (TurboFan in V8)**:
  - Profiles running code to detect "hot functions" (frequently executed code paths).
  - Optimizes bytecode into machine code using assumptions.
  - **Deoptimization**: If assumptions fail (e.g., hidden class changes because a property type changed), code deoptimizes back to bytecode.
- [ ] **Hidden Classes & Inline Caches**:
  - Objects that share the same shape (same keys added in the same order) share hidden classes.
  - *Best Practice*: Always initialize object properties in the same order and avoid deleting properties with `delete` (use `null` or `undefined` instead).

### Execution Context (EC)
- [ ] Everything in JavaScript runs inside an **Execution Context**.
- [ ] **Global Execution Context (GEC)**: Created when the script begins execution; only one GEC exists.
- [ ] **Function Execution Context (FEC)**: Created every time a function is called.
- [ ] **Eval Execution Context**: Created when code runs inside `eval()`.
- [ ] **Two Phases of Execution Context**:
  1. **Creation Phase (Memory Allocation)**:
     - Global object (`window` or `globalThis`) is created.
     - `this` binding is established.
     - Memory allocated for variables and functions:
       - `var` $\to$ initialized to `undefined`.
       - `let` and `const` $\to$ uninitialized (Temporal Dead Zone).
       - Function declarations $\to$ stored in memory with their complete implementation.
  2. **Execution Phase**:
     - Executes line-by-line; variables assigned their concrete values.

---

## 2. Memory Management & Garbage Collection

### Memory Heap & Call Stack
- [ ] **Call Stack**: Fixed size, holds primitive types and execution context stack frames. High speed, LIFO.
- [ ] **Memory Heap**: Unstructured memory space for dynamic object allocation, arrays, and functions.

### Garbage Collection (Mark-and-Sweep)
- [ ] **Root Objects**: Global variables, currently running function execution contexts, active DOM trees.
- [ ] **Algorithm**:
  1. The GC starts at roots and marks all objects directly referenced by them.
  2. Follows references recursively, marking all reachable objects.
  3. Sweeps unreferenced memory and returns it to the free memory pool.
- [ ] **Common Memory Leaks**:
  1. *Accidental Globals*: Assigning to undeclared variables (`foo = "bar"` attaches to `window`).
  2. *Dangling Timers*: `setInterval` callback holding onto outer scope references long after the component is gone.
  3. *Detached DOM Nodes*: A JS variable references a DOM element that was removed from the document (`document.body.removeChild()`).
  4. *Uncleared Event Listeners*: Not removing event listeners on `window` or `document` when a view unmounts.
  5. *Closure Retention*: Large data structures trapped inside long-lived closures.

---

## 3. Scopes, Scope Chain & Closures

### Scopes
- [ ] **Global Scope**: Accessible everywhere.
- [ ] **Function / Local Scope**: Defined inside a function body.
- [ ] **Block Scope**: Introduced in ES6 (`let` and `const` inside `{ ... }`).
- [ ] **Lexical Scope (Static Scope)**: Scope resolution is determined by the physical placement of variables and blocks in source code at author time, not where functions are invoked.

### Closures
> A closure is the combination of a function bundled together with references to its surrounding state (lexical environment).

- [ ] **Use Cases**:
  - Data privacy / Encapsulation (Module pattern).
  - Function Currying & Partial Application.
  - Memoization & caching.
  - Event handlers retaining component state.
- [ ] **Classic Loop Trap**:
  ```javascript
  // Problem with var (function scoped)
  for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // 3, 3, 3
  }

  // Solution 1: let (block scoped)
  for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // 0, 1, 2
  }

  // Solution 2: IIFE (creates separate function scope)
  for (var i = 0; i < 3; i++) {
    ((idx) => {
      setTimeout(() => console.log(idx), 100); // 0, 1, 2
    })(i);
  }
  ```

---

## 4. Hoisting & The Temporal Dead Zone (TDZ)

- [ ] `var`: Hoisted to the top of its function or global scope and initialized to `undefined`.
- [ ] `let` and `const`: Hoisted to the top of their block scope, but remain **uninitialized** in the Temporal Dead Zone (TDZ). Accessing them before their declaration line throws a `ReferenceError`.
- [ ] Function Declarations vs Expressions:
  ```javascript
  greet(); // "Hello!" (Function declaration hoisted with body)
  function greet() { console.log("Hello!"); }

  sayHi(); // TypeError: sayHi is not a function (sayHi is hoisted as undefined)
  var sayHi = function() { console.log("Hi!"); };
  ```

---

## 5. The `this` Keyword & Explicit Binding

### 4 Rules of `this` Determination (in order of priority)
1. **`new` Binding**: When called with `new`, `this` refers to the newly constructed instance.
2. **Explicit Binding**: Using `.call()`, `.apply()`, or `.bind()`.
   - `fn.call(ctx, arg1, arg2)`
   - `fn.apply(ctx, [arg1, arg2])`
   - `const boundFn = fn.bind(ctx, arg1)` (returns a new function permanently bound to `ctx`).
3. **Implicit Binding**: When invoked as a method of an object (`obj.method()`), `this` refers to `obj`.
4. **Default Binding**: Standalone function invocation (`fn()`). In non-strict mode, `this` is `window`/`global`; in strict mode (`'use strict'`), `this` is `undefined`.

### Arrow Functions
- [ ] Do **not** have their own `this`, `arguments`, `super`, or `new.target`.
- [ ] They resolve `this` lexically from the enclosing lexical scope at declaration time.
- [ ] Calling `.call()`, `.apply()`, or `.bind()` on an arrow function has **no effect** on its `this`.

---

## 6. Prototypes & Prototypal Inheritance

- [ ] Every JS object has a hidden link to its prototype (`[[Prototype]]`, accessible via `Object.getPrototypeOf()` or `__proto__`).
- [ ] Functions have a `.prototype` property that becomes the `[[Prototype]]` of objects created when calling that function with `new`.
- [ ] Prototype chain terminal: `Object.prototype.__proto__ === null`.
- [ ] **Inheritance Patterns**:
  ```javascript
  function Animal(name) {
    this.name = name;
  }
  Animal.prototype.speak = function() {
    return `${this.name} makes a sound.`;
  };

  function Dog(name, breed) {
    Animal.call(this, name); // Call super constructor
    this.breed = breed;
  }
  // Link prototypes
  Dog.prototype = Object.create(Animal.prototype);
  Dog.prototype.constructor = Dog;

  Dog.prototype.bark = function() {
    return `${this.name} barks!`;
  };
  ```
- [ ] ES6 `class` syntax is sugar over this prototype mechanism:
  - Methods defined on `class` are placed on `ClassName.prototype`.
  - Static methods are placed directly on the constructor function `ClassName`.

---

## 7. Asynchronous JavaScript & The Event Loop

### Components of the Concurrency Model
1. **Call Stack**: Executes synchronous JS single-threadedly.
2. **Web APIs / Libuv**: Browser or Node threads executing timer countdowns, HTTP requests, file reads.
3. **Microtask Queue** (Highest Priority):
   - `Promise.then / catch / finally`
   - `queueMicrotask()`
   - `MutationObserver`
   - `process.nextTick` (Node.js)
4. **Macrotask Queue (Task Queue)**:
   - `setTimeout`, `setInterval`, `setImmediate` (Node), DOM events, I/O callbacks.
5. **Animation Frame Queue**:
   - `requestAnimationFrame` (runs before browser paint).

### Event Loop Turn (Tick) Rule:
```
1. Run one Macrotask from the Task Queue (or initial script).
2. Drain ALL Microtasks in the Microtask Queue until empty.
   (If microtasks enqueue more microtasks, they are executed in the same turn).
3. If repaint is needed, run requestAnimationFrame callbacks and render UI.
4. Pick the next Macrotask and repeat.
```

---

## 8. Promises, Async/Await & Microtask Scheduling

### Promise States & Transitions
- `pending` $\to$ `fulfilled` (with a value)
- `pending` $\to$ `rejected` (with an error/reason)
- State transitions are immutable: once settled, a promise can never change state.

### Promise Combinators
- [ ] `Promise.all([p1, p2])`: Fails fast if ANY promise rejects. Resolves with array of values if all fulfill.
- [ ] `Promise.allSettled([p1, p2])`: Always waits for all promises to settle; returns array of `{ status: 'fulfilled' | 'rejected', value | reason }`.
- [ ] `Promise.race([p1, p2])`: Settles as soon as the first promise settles (fulfills or rejects).
- [ ] `Promise.any([p1, p2])`: Resolves as soon as the first promise fulfills. Rejects only if all reject (`AggregateError`).

### Async/Await Mechanics
- `async` functions always return a Promise.
- `await` pauses execution of the async function body until the awaited promise settles, yielding control back to the event loop.

---

## 9. ES6+ Modern Features & Advanced Metaprogramming

- [ ] **Destructuring & Rest/Spread**: Deep destructuring, default values, array/object cloning.
- [ ] **Symbols**: Unique, immutable primitives used for private-like properties and well-known symbols (`Symbol.iterator`, `Symbol.toPrimitive`, `Symbol.hasInstance`).
- [ ] **Generators & Iterators**:
  - Protocols: An iterable implements `[Symbol.iterator]()` which returns an object with a `next()` method returning `{ value, done }`.
  - `function* gen()` with `yield`.
- [ ] **Map & Set vs Object & Array**:
  - `Map` supports keys of any type (including objects/functions), maintains insertion order, has `.size`.
  - `Set` stores unique values.
  - `WeakMap` & `WeakSet`: Keys must be objects; holds weak references allowing garbage collection when no other references exist; non-enumerable.
- [ ] **Proxy & Reflect**:
  - `Proxy` intercepts low-level operations (get, set, has, deleteProperty, apply).
  - Used in modern reactive libraries (e.g., Vue 3 reactivity).

---

## 10. Core Polyfills & Essential Utilities

### 1. `Function.prototype.bind` Polyfill
```javascript
Function.prototype.myBind = function(context, ...boundArgs) {
  if (typeof this !== 'function') {
    throw new TypeError('Not callable');
  }
  const originalFn = this;

  return function bound(...callArgs) {
    // Check if called as a constructor with 'new'
    if (this instanceof bound) {
      return new originalFn(...boundArgs, ...callArgs);
    }
    return originalFn.apply(context, boundArgs.concat(callArgs));
  };
};
```

### 2. `Array.prototype.flat` Polyfill
```javascript
Array.prototype.myFlat = function(depth = 1) {
  return depth > 0
    ? this.reduce((acc, val) => {
        return acc.concat(Array.isArray(val) ? val.myFlat(depth - 1) : val);
      }, [])
    : this.slice();
};
```

### 3. `debounce` Implementation
```javascript
function debounce(fn, delay) {
  let timerId = null;

  function debounced(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  }

  debounced.cancel = () => clearTimeout(timerId);
  return debounced;
}
```

### 4. `throttle` Implementation
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

### 5. `Promise.all` Polyfill
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }
    const results = [];
    let completed = 0;
    const total = promises.length;

    if (total === 0) return resolve(results);

    promises.forEach((p, index) => {
      Promise.resolve(p)
        .then((val) => {
          results[index] = val;
          completed++;
          if (completed === total) resolve(results);
        })
        .catch(reject);
    });
  });
}
```

---

## 11. Tricky JS Output Questions & Gotchas

### Output Quiz 1: Event Loop Order
```javascript
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
queueMicrotask(() => console.log('D'));
console.log('E');
// Order: A, E, C, D, B
```

### Output Quiz 2: `this` Context
```javascript
const obj = {
  name: 'Antigravity',
  regular() { return this.name; },
  arrow: () => this.name,
};

console.log(obj.regular()); // "Antigravity"
console.log(obj.arrow());   // undefined (or window.name in browser)

const fn = obj.regular;
console.log(fn());          // undefined (loss of implicit binding)
```

### Output Quiz 3: Object Key Coercion
```javascript
const a = {}, b = { key: 'b' }, c = { key: 'c' };
a[b] = 123;
a[c] = 456;
console.log(a[b]); // 456 (both b and c stringify to "[object Object]")
```
