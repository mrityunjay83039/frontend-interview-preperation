# 📐 Frontend System Design & Architecture Interview Guide

A battle-tested framework for tackling senior and staff-level Frontend System Design interviews.

---

## 📑 Table of Contents
1. [The 4-Step System Design Interview Framework](#1-the-4-step-system-design-interview-framework)
2. [Data Modeling & Client State Architecture](#2-data-modeling--client-state-architecture)
3. [Networking, Real-Time Sync & Caching](#3-networking-real-time-sync--caching)
4. [Case Study 1: Autocomplete / Typeahead Search](#4-case-study-1-autocomplete--typeahead-search)
5. [Case Study 2: Infinite Scroll & Virtualized Feed](#5-case-study-2-infinite-scroll--virtualized-feed)
6. [Case Study 3: Real-Time Collaborative Document / Chat](#6-case-study-3-real-time-collaborative-document--chat)
7. [Case Study 4: Multi-Step Form with Offline Support](#7-case-study-4-multi-step-form-with-offline-support)
8. [Design Systems & Micro-Frontends](#8-design-systems--micro-frontends)

---

## 1. The 4-Step System Design Interview Framework

In a 45-60 minute interview, structure your time strictly:

### Step 1: Requirements Gathering & Scoping (5-8 mins)
- **Functional Requirements**: What actions can the user perform? (e.g. search, select, paginate, edit offline).
- **Non-Functional Requirements**:
  - Performance: INP $\le 100\text{ms}$, 60fps scrolling, minimal bundle size.
  - Scale: Support 100k items without freezing DOM.
  - Device/Network: Responsive mobile support, slow 3G network resilience.
  - Accessibility: WCAG AA compliant, full keyboard navigation.

### Step 2: High-Level Component & State Architecture (10-15 mins)
- Draw the component tree (Container vs Presentational components).
- Define client state store schema (normalized structures).
- Define API contracts (REST, GraphQL, WebSockets) and payload shapes.

### Step 3: Deep Dive into Core Challenges (15-20 mins)
- Pick 2-3 complex technical areas (e.g., list virtualization, optimistic UI updates, conflict resolution, offline synchronization).
- Explain data flow diagrams and algorithms.

### Step 4: Edge Cases, Performance & Security (5-10 mins)
- Error handling, offline mode, rate limiting, token expiration, accessibility focus traps, XSS sanitization.

---

## 2. Data Modeling & Client State Architecture

### Normalized State Pattern
Never store nested relational arrays directly in client state. Normalize using the `byId` and `allIds` pattern:

```typescript
// Un-normalized (BAD: Hard to update, duplicate entities)
const state = {
  posts: [
    { id: 'p1', author: { id: 'u1', name: 'Alice' }, comments: [{ id: 'c1', text: 'Great!' }] }
  ]
};

// Normalized (GOOD: Single source of truth, O(1) lookups)
interface NormalizedState {
  users: {
    byId: Record<string, { id: string; name: string }>;
  };
  comments: {
    byId: Record<string, { id: string; text: string; authorId: string }>;
  };
  posts: {
    byId: Record<string, { id: string; authorId: string; commentIds: string[] }>;
    allIds: string[];
  };
}
```

---

## 3. Networking, Real-Time Sync & Caching

### Communication Protocols
| Protocol | Directionality | Overhead | Best For |
| :--- | :--- | :--- | :--- |
| **Short Polling** | Unidirectional (Client $\to$ Server) | Very High (repeated HTTP handshakes) | Simple prototyping |
| **Long Polling** | Unidirectional (Server holds connection open) | High | Fallback environments where WebSockets are blocked |
| **Server-Sent Events (SSE)** | Unidirectional (Server $\to$ Client) | Low (single persistent HTTP connection) | Stock tickers, live sports scores, notification feeds |
| **WebSockets** | Full Duplex (Bidirectional) | Minimal per message | Collaborative editing, real-time multiplayer games, active chat |

### Optimistic UI Updates with Rollback
1. User clicks "Like".
2. Immediately mutate local UI state (optimistic increment).
3. Send network request in background.
4. If request succeeds: silent confirmation.
5. If request fails: rollback to previous snapshot and display toast error with retry button.

---

## 4. Case Study 1: Autocomplete / Typeahead Search

```
User Types ──> [Debounce 300ms] ──> [Check In-Memory Cache] ──(Hit)──> Render Results
                                                │ (Miss)
                                                ▼
                                    [AbortController Abort Old]
                                                │
                                                ▼
                                        [Fetch New Query] ──> Store in Cache & Render
```

### Key Technical Challenges
1. **Race Conditions**: Network requests can return out of order.
   - *Solution*: Use `AbortController` to cancel pending in-flight requests when the user types a new keystroke.
2. **High Network Traffic**:
   - *Solution*: Debounce keystrokes (250-300ms), implement client-side LRU cache in memory.
3. **Keyboard Accessibility**:
   - Up/Down arrows traverse items using `aria-activedescendant`.
   - `Enter` selects active item; `Escape` closes popup.

---

## 5. Case Study 2: Infinite Scroll & Virtualized Feed

### The Problem with 10,000 DOM Nodes
Rendering thousands of cards causes excessive memory consumption, slow scroll FPS, and slow GC pauses.

### Virtualization (Windowing) Algorithm
- Only render items currently visible inside the viewport, plus a buffer of 3-5 items above and below.
- Use absolute positioning with `transform: translateY(offsetY)` or top/bottom spacer `<div>`s to maintain the true scrollbar height.

```
┌─────────────────────────────────┐ ◄── Total Container Height (e.g. 100,000px)
│        Top Spacer DIV           │
├─────────────────────────────────┤ ◄── Viewport Top
│      [Rendered Item 1]          │
│      [Rendered Item 2]          │ ◄── Viewport Active Visible Window
│      [Rendered Item 3]          │
├─────────────────────────────────┤ ◄── Viewport Bottom
│       Bottom Spacer DIV         │
└─────────────────────────────────┘
```

- **Pagination**: Use cursor-based pagination (`next_cursor=timestamp_id`) rather than offset pagination to avoid skipping or duplicating posts when new content is inserted at the top.

---

## 6. Case Study 3: Real-Time Collaborative Document / Chat

### Key Architecture
- WebSocket connection with heartbeat ping/pong to detect dropped links.
- Client message queue storing messages with local UUIDs.
- Optimistic states: `sending` $\to$ `sent` $\to$ `delivered` $\to$ `failed`.
- Conflict resolution:
  - Simple: Last-Write-Wins (LWW) with timestamp.
  - Complex collaborative text: Operational Transformation (OT) or CRDTs (Yjs, Automerge).

---

## 7. Case Study 4: Multi-Step Form with Offline Support

- Store form progress in `IndexedDB` or `localStorage` on every field change.
- Service Worker caching form static shell.
- Background Sync API to automatically flush submitted data once the user regains internet connectivity.

---

## 8. Design Systems & Micro-Frontends

- **Design System Architecture**:
  - Design Tokens (JSON for colors, typography, spacing).
  - Headless UI primitives (Radix, React Aria) providing accessibility and keyboard handling without styling.
  - Styled layer (Tailwind or CSS modules).
- **Micro-Frontends**:
  - Module Federation (Webpack 5): Dynamically loads remote components at runtime across different deployments.
  - Independent versioning and deployment pipelines per domain team.
