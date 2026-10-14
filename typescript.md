# 🔷 TypeScript Comprehensive Interview Guide

A structured guide covering TypeScript's type system, type theory, advanced generics, utility types, and patterns used in production frontend development.

---

## 📑 Table of Contents
1. [Core Type System & Primitives](#1-core-type-system--primitives)
2. [`type` vs `interface`](#2-type-vs-interface)
3. [Type Narrowing & Control Flow Analysis](#3-type-narrowing--control-flow-analysis)
4. [Discriminated Unions](#4-discriminated-unions)
5. [Generics & Constraints](#5-generics--constraints)
6. [Conditional & Mapped Types](#6-conditional--mapped-types)
7. [Built-in Utility Types Polyfilled & Explained](#7-built-in-utility-types-polyfilled--explained)
8. [TypeScript with React](#8-typescript-with-react)
9. [Common TS Interview Gotchas](#9-common-ts-interview-gotchas)

---

## 1. Core Type System & Primitives

### `unknown` vs `any` vs `never`
- [ ] **`any`**: Opts out of all type checking completely. Can be assigned to anything and anything can be assigned to it.
- [ ] **`unknown`**: Top type (safe counterpart of `any`). Anything can be assigned to `unknown`, but `unknown` cannot be used or assigned to other types without explicit **type narrowing** or casting.
- [ ] **`never`**: Bottom type. Represents the type of values that never occur.
  - Examples: Function that always throws an error, infinite loop, or the remaining branch of an exhaustive switch statement.

```typescript
function assertNever(x: never): never {
  throw new Error(`Unexpected object: ${x}`);
}
```

### `void` vs `undefined`
- `void`: Indicates a function does not return a meaningful value.
- `undefined`: The literal primitive value in JS.

---

## 2. `type` vs `interface`

| Feature | `interface` | `type` |
| :--- | :--- | :--- |
| **Declaration Merging** | ✅ Yes (merges multiple interfaces with same name) | ❌ No (duplicate identifier error) |
| **Unions & Intersections** | ❌ Cannot directly express a union | ✅ Yes (`type Status = 'open' \| 'closed'`) |
| **Primitives & Tuples** | ❌ Cannot alias primitive values | ✅ Yes (`type ID = string \| number`) |
| **Extending / Inheritance** | `interface B extends A` | `type B = A & { extra: string }` |
| **Best Practice** | Use for public API models, OOP contracts, component props | Use for unions, complex mapped types, tuples |

---

## 3. Type Narrowing & Control Flow Analysis

- [ ] **`typeof` guard**: Narrow primitives (`string`, `number`, `boolean`, `symbol`).
- [ ] **`instanceof` guard**: Narrow class instances and built-in objects (`Date`, `Error`, `RegExp`).
- [ ] **`in` operator guard**: Check for existence of a property on an object.
- [ ] **Custom Type Guards (`is` predicate)**:
  ```typescript
  interface Fish { swim: () => void; }
  interface Bird { fly: () => void; }

  function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
  }

  function move(pet: Fish | Bird) {
    if (isFish(pet)) {
      pet.swim(); // TS knows pet is Fish
    } else {
      pet.fly();  // TS knows pet is Bird
    }
  }
  ```

---

## 4. Discriminated Unions

A pattern where multiple object types share a common literal field (the discriminant):

```typescript
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: string[] };
type ErrorState = { status: 'error'; message: string };

type ViewState = LoadingState | SuccessState | ErrorState;

function renderView(state: ViewState) {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    case 'success':
      return `Loaded ${state.data.length} items`;
    case 'error':
      return `Error: ${state.message}`;
    default:
      // Exhaustiveness check
      const _exhaustiveCheck: never = state;
      return _exhaustiveCheck;
  }
}
```

---

## 5. Generics & Constraints

Generics allow building reusable, type-safe components and functions:

```typescript
// Generic with constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Alice' };
const name = getProperty(user, 'name'); // Typed as string
// getProperty(user, 'email'); // Compile Error: Argument of type '"email"' is not assignable to keyof user
```

---

## 6. Conditional & Mapped Types

### Conditional Types
`T extends U ? X : Y`
```typescript
type IsString<T> = T extends string ? true : false;
type A = IsString<string>; // true
type B = IsString<number>; // false
```

### Mapped Types
```typescript
type Optional<T> = {
  [K in keyof T]?: T[K];
};
```

---

## 7. Built-in Utility Types Explained & Implemented

### 1. `Partial<T>`
```typescript
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};
```

### 2. `Required<T>`
```typescript
type MyRequired<T> = {
  [K in keyof T]-?: T[K];
};
```

### 3. `Pick<T, K>`
```typescript
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

### 4. `Omit<T, K>`
```typescript
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

### 5. `Record<K, T>`
```typescript
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};
```

### 6. `ReturnType<T>`
```typescript
type MyReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;
```

---

## 8. TypeScript with React

### Typing React Component Props
```typescript
import React, { ReactNode } from 'react';

type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  children: ReactNode;
  onClick: (e: React.MouseEvent<HTMLButtonElement>) => void;
  disabled?: boolean;
} & React.ComponentPropsWithoutRef<'button'>;

export const Button: React.FC<ButtonProps> = ({ variant = 'primary', children, ...rest }) => {
  return <button className={`btn-${variant}`} {...rest}>{children}</button>;
};
```

---

## 9. Common TS Interview Gotchas

- [ ] **Structural Typing vs Nominal Typing**: TypeScript is structurally typed (duck typing). Two types are compatible if their members are compatible, regardless of explicit declaration.
- [ ] **Covariance vs Contravariance**: Function return types are covariant; function argument types are contravariant.
- [ ] **`const` assertions**:
  ```typescript
  const routes = ['home', 'about', 'contact'] as const;
  type Route = typeof routes[number]; // "home" | "about" | "contact"
  ```
