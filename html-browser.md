# 🌐 HTML5, Browser Internals & Web APIs Interview Guide

A comprehensive guide to how browsers work under the hood, DOM manipulation, Web APIs, storage mechanisms, and web security.

---

## 📑 Table of Contents
1. [The Critical Rendering Path (CRP)](#1-the-critical-rendering-path-crp)
2. [Reflow (Layout) vs Repaint (Paint)](#2-reflow-layout-vs-repaint-paint)
3. [Script Loading Strategies & Resource Hints](#3-script-loading-strategies--resource-hints)
4. [DOM Events, Bubbling, Capturing & Delegation](#4-dom-events-bubbling-capturing--delegation)
5. [Browser Storage Mechanisms](#5-browser-storage-mechanisms)
6. [Essential Modern Web APIs](#6-essential-modern-web-apis)
7. [Web Security: CORS, XSS, CSRF & CSP](#7-web-security-cors-xss-csrf--csp)
8. [Semantic HTML & SEO Best Practices](#8-semantic-html--seo-best-practices)

---

## 1. The Critical Rendering Path (CRP)

The sequential steps the browser executes to turn HTML, CSS, and JavaScript into active pixels on the screen:

```
HTML Bytes ──> Characters ──> Tokens ──> Nodes ──> DOM Tree
                                                        ├──> Render Tree ──> Layout ──> Paint ──> Composite
CSS Bytes  ──> Characters ──> Tokens ──> Nodes ──> CSSOM Tree
```

1. **DOM Construction**: HTML parser reads bytes, tokenizes tags, and constructs the DOM tree. HTML parsing is incremental.
2. **CSSOM Construction**: CSS is **render-blocking**. The browser cannot build the Render Tree until the entire CSSOM is fully built because CSS rules cascade.
3. **Render Tree**: Combines DOM and CSSOM. Nodes with `display: none` or `<head>` / `<script>` are omitted. Nodes with `visibility: hidden` are included (take up space).
4. **Layout (Reflow)**: Computes the exact geometric dimensions and screen position of every visible element.
5. **Paint (Repaint)**: Translates layout geometry into actual colors, pixels, borders, and shadows on layers.
6. **Compositing**: Individual layers are uploaded to the GPU and rasterized onto the screen.

---

## 2. Reflow (Layout) vs Repaint (Paint)

### Reflow Triggers
- Adding, removing, or reordering DOM nodes.
- Modifying geometry properties: `width`, `height`, `padding`, `margin`, `border`, `top`, `fontSize`.
- Resizing the browser window or changing orientation.
- **Layout Thrashing (Forced Synchronous Layout)**: Reading layout properties immediately after writing:
  ```javascript
  // BAD: Layout Thrashing (forces recalculation on every iteration)
  elements.forEach(el => {
    const width = el.offsetWidth; // READ forces reflow
    el.style.width = width + 10 + 'px'; // WRITE invalidates layout
  });

  // GOOD: Batch reads, then batch writes
  const widths = elements.map(el => el.offsetWidth);
  elements.forEach((el, i) => {
    el.style.width = widths[i] + 10 + 'px';
  });
  ```

### Repaint Triggers
- Altering visual styles that don't affect layout geometry: `color`, `background-color`, `box-shadow`, `visibility`.

### GPU-Accelerated Compositing (Best Performance)
- Modifying `transform` (`translate3d`, `scale`) or `opacity` bypasses both Reflow and Repaint.
- Runs directly on GPU layers at 60/120fps.

---

## 3. Script Loading Strategies & Resource Hints

### Script Tag Loading Behavior
- `<script src="app.js">`: Blocks HTML parser. HTML parsing pauses, script is downloaded and executed immediately, then parsing resumes.
- `<script async src="app.js">`: Fetches asynchronously in parallel with HTML parsing. As soon as download completes, HTML parsing is paused to execute the script immediately. Order of execution is non-deterministic.
- `<script defer src="app.js">`: Fetches asynchronously in parallel with HTML parsing. Script execution is deferred until HTML parsing completes (guaranteed execution in document order right before `DOMContentLoaded`).
- `<script type="module" src="app.js">`: Defer behavior by default, runs in strict mode, supports ES modules.

### Resource Hints
- **`dns-prefetch`**: Resolves DNS before a user clicks an outbound link or fetches a third-party asset.
- **`preconnect`**: DNS + TCP handshake + TLS negotiation with an origin (e.g., Google Fonts, CDN).
- **`preload`**: Mandatory high-priority fetch for critical assets needed on the current page (e.g., LCP hero image, critical web font).
- **`prefetch`**: Low-priority fetch for resources likely needed on subsequent navigations.

---

## 4. DOM Events, Bubbling, Capturing & Delegation

### The 3 Event Phases
```
                 WINDOW
                   │
           [1. Capturing Phase]
                   ▼
               DOCUMENT
                   ▼
                 BODY
                   ▼
              DIV CONTAINER
                   ▼
            [2. Target Phase] ──> BUTTON
                   │
           [3. Bubbling Phase]
                   ▼
              DIV CONTAINER
                   ▼
                 BODY
                   ▼
                 WINDOW
```

- `addEventListener(event, handler, useCapture)`:
  - `useCapture = false` (default): Triggered during Bubbling.
  - `useCapture = true`: Triggered during Capturing.
- `e.target`: The actual element that initiated the event (e.g., the clicked `<button>`).
- `e.currentTarget`: The element handling the event (where `addEventListener` was registered).
- `e.stopPropagation()`: Stops propagation up or down the DOM tree.
- `e.preventDefault()`: Prevents the browser's default action (e.g., following an `<a href>` or submitting a `<form>`).

### Event Delegation
Registering one handler on a parent node to manage clicks for many current and future child items:
```javascript
document.querySelector('#todo-list').addEventListener('click', (e) => {
  const item = e.target.closest('.todo-item');
  if (item) {
    toggleTodo(item.dataset.id);
  }
});
```

---

## 5. Browser Storage Mechanisms

| Storage | Capacity | Expiry | Sent with HTTP Requests? | Scope | Typical Usage |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LocalStorage** | 5 - 10 MB | Never (until manual clear) | No | Origin | UI theme, offline drafts |
| **SessionStorage** | ~5 MB | Tab close | No | Origin & Tab | Checkout wizards, temp forms |
| **Cookies** | ~4 KB | Configurable date | Yes (all requests to origin/path) | Origin / Domain | Auth tokens (`HttpOnly`), session tracking |
| **IndexedDB** | >50 MB (quota) | Persistent | No | Origin | Large offline databases, PWA caching |

### Cookie Security Flags
- `HttpOnly`: Prevents client-side JS from accessing the cookie (`document.cookie`), mitigating XSS token theft.
- `Secure`: Cookie is only transmitted over HTTPS connections.
- `SameSite=Strict`: Never sent with cross-site requests (protects against CSRF).
- `SameSite=Lax`: Sent with top-level navigations (safe links), protects against most CSRF attacks.

---

## 6. Essential Modern Web APIs

### 1. `IntersectionObserver`
Detects when an element enters or leaves the viewport (ideal for image lazy-loading, infinite scrolling, tracking ad visibility):
```javascript
const observer = new IntersectionObserver((entries, obs) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      obs.unobserve(img); // Stop observing once loaded
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

### 2. `AbortController`
Cancels ongoing `fetch` requests or removes event listeners:
```javascript
const controller = new AbortController();

fetch('/api/search?q=test', { signal: controller.signal })
  .then(res => res.json())
  .catch(err => {
    if (err.name === 'AbortError') console.log('Request aborted');
  });

// Cancel if user types another query:
controller.abort();
```

### 3. `MutationObserver`
Monitors changes to the DOM tree (attributes, child nodes, character data).

### 4. Web Workers
Executes CPU-intensive calculations on background threads without blocking the main browser UI thread.

---

## 7. Web Security: CORS, XSS, CSRF & CSP

### Cross-Origin Resource Sharing (CORS)
- **Same-Origin Policy (SOP)**: Disallows reading cross-origin responses unless the server explicitly grants permission via HTTP response headers:
  - `Access-Control-Allow-Origin: https://myapp.com`
  - `Access-Control-Allow-Methods: GET, POST, PUT, DELETE`
  - `Access-Control-Allow-Headers: Content-Type, Authorization`
- **Preflight Request**: Browser issues an `OPTIONS` request before non-simple HTTP requests (e.g., custom headers, JSON `Content-Type`, `PUT`/`DELETE` methods).

### Cross-Site Scripting (XSS)
- Injection of malicious client-side JavaScript.
- **Stored XSS**: Malicious script saved in database (e.g. comment field) and rendered unsanitized to other users.
- **Reflected XSS**: Script injected via URL parameters.
- **DOM-based XSS**: Vulnerable client-side JS directly sets `element.innerHTML = location.hash`.
- **Defenses**: Context-aware escaping, DOMPurify, Content Security Policy (`default-src 'self'`), `HttpOnly` cookies.

### Cross-Site Request Forgery (CSRF)
- Deceiving an authenticated browser into submitting an unauthorized state-changing request.
- **Defenses**: `SameSite=Lax/Strict` cookies, Anti-CSRF challenge tokens, `Origin` / `Referer` validation.

---

## 8. Semantic HTML & SEO Best Practices

- [ ] Use semantic structure: `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`.
- [ ] Exactly one `<h1>` per page.
- [ ] Meaningful `<title>` and `<meta name="description">` tags.
- [ ] Open Graph (`og:title`, `og:image`) and Twitter card tags.
- [ ] Descriptive alt text for all informative images (`<img alt="User dashboard preview">`).
