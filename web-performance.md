# ⚡ Web Performance & Core Web Vitals Interview Guide

A comprehensive guide to measuring, diagnosing, and optimizing web application performance across the network, browser runtime, and rendering engines.

---

## 📑 Table of Contents
1. [Core Web Vitals (LCP, INP, CLS)](#1-core-web-vitals-lcp-inp-cls)
2. [Network Layer Optimizations (HTTP/2, HTTP/3, CDN)](#2-network-layer-optimizations-http2-http3-cdn)
3. [HTTP Caching & Headers](#3-http-caching--headers)
4. [JavaScript Bundle Optimization & Tree Shaking](#4-javascript-bundle-optimization--tree-shaking)
5. [Asset & Media Optimization](#5-asset--media-optimization)
6. [Rendering Architecture Tradeoffs (CSR, SSR, SSG, ISR)](#6-rendering-architecture-tradeoffs-csr-ssr-ssg-isr)
7. [Diagnosing & Profiling Tools](#7-diagnosing--profiling-tools)

---

## 1. Core Web Vitals (LCP, INP, CLS)

The Google Core Web Vitals determine real-world user experience and directly affect search engine rankings.

| Metric | Name | Good | Needs Improvement | Poor | What It Measures |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LCP** | Largest Contentful Paint | $\le 2.5\text{s}$ | $2.5\text{s} - 4.0\text{s}$ | $> 4.0\text{s}$ | Loading speed of the primary visible content |
| **INP** | Interaction to Next Paint | $\le 200\text{ms}$ | $200\text{ms} - 500\text{ms}$ | $> 500\text{ms}$ | Overall page responsiveness to user inputs (clicks, keypresses) |
| **CLS** | Cumulative Layout Shift | $\le 0.1$ | $0.1 - 0.25$ | $> 0.25$ | Visual stability (unexpected layout shifting) |

### 1. Largest Contentful Paint (LCP) Optimization
- **Identify the LCP element**: Usually the hero image, banner video, or large header text block.
- **Remediations**:
  1. Add `fetchpriority="high"` and `<link rel="preload">` for the hero image:
     ```html
     <link rel="preload" as="image" href="/hero.webp" fetchpriority="high">
     ```
  2. Avoid lazy-loading the hero image (`loading="lazy"` on LCP elements severely hurts LCP).
  3. Serve images in modern formats (AVIF, WebP) with responsive `<picture>` tags.
  4. Use a Content Delivery Network (CDN) with edge caching to reduce TTFB (Time to First Byte).

### 2. Interaction to Next Paint (INP) Optimization
*Replaced FID (First Input Delay) in March 2024.*
- Measures the latency of **every** interaction throughout the entire lifecycle of the page, reporting the worst (98th percentile).
- **Remediations**:
  1. Break long tasks ($> 50\text{ms}$) on the main thread using `scheduler.yield()` or `setTimeout(0)`.
  2. Debounce high-frequency events (search input, window resize).
  3. Move heavy calculations to Web Workers.
  4. Avoid layout thrashing in event handlers.

### 3. Cumulative Layout Shift (CLS) Optimization
- Measures the fraction of the viewport that shifts unexpectedly while the page loads.
- **Remediations**:
  1. Always set explicit `width` and `height` attributes (or CSS `aspect-ratio`) on images and video embeds.
  2. Reserve space for dynamic content, ads, and banners using skeleton screens or placeholder boxes.
  3. Use `font-display: swap` or `optional` combined with size-adjusted fallback fonts (`size-adjust`, `ascent-override`) to avoid Flash of Unstyled Text (FOUT) layout jumps.

---

## 2. Network Layer Optimizations (HTTP/2, HTTP/3, CDN)

### Protocol Comparison
- **HTTP/1.1**:
  - Head-of-line blocking at application layer.
  - Browser creates 6 simultaneous TCP connections per domain.
  - Requires domain sharding and CSS spriting to optimize.
- **HTTP/2**:
  - Single TCP connection with binary framing.
  - **Multiplexing**: Multiple requests and responses interleaved over one connection.
  - Header compression (HPACK).
- **HTTP/3**:
  - Built on **QUIC** protocol running over **UDP**.
  - Solves TCP head-of-line blocking: If one packet drops, only that specific stream is stalled, not the entire connection.
  - Zero-RTT connection establishment on reconnects.

---

## 3. HTTP Caching & Headers

### Cache Directives
```http
Cache-Control: public, max-age=31536000, immutable
```
- **Hashed Assets (e.g. `main.a9f4c.js`)**: Cache forever (`max-age=31536000, immutable`). When code changes, the build tool changes the filename hash.
- **`index.html`**: Never cache or require revalidation:
  ```http
  Cache-Control: no-cache
  ```
  *(Forces browser to check server with `ETag` / `If-None-Match` before serving; returns `304 Not Modified` if unchanged).*
- **`no-store`**: Never store in any cache (for private user data).
- **`stale-while-revalidate`**:
  ```http
  Cache-Control: max-age=60, stale-while-revalidate=600
  ```
  *Serves cached version instantly while revalidating in background for future requests.*

---

## 4. JavaScript Bundle Optimization & Tree Shaking

- [ ] **Tree Shaking**: Eliminates unused exports from bundles. Requires ES module syntax (`import`/`export`) and `"sideEffects": false` in `package.json`.
- [ ] **Dynamic Code Splitting**:
  ```javascript
  // Route-based or interaction-based splitting
  const Chart = await import('./HeavyChartComponent');
  ```
- [ ] **Analyzing Bundles**: Use `webpack-bundle-analyzer`, `rollup-plugin-visualizer`, or Vite's bundle analyzer to identify bloated libraries (e.g., replacing `moment.js` with `date-fns` or native `Intl`).

---

## 5. Asset & Media Optimization

- [ ] **Responsive Images**:
  ```html
  <picture>
    <source srcset="/img-800.avif 800w, /img-1200.avif 1200w" type="image/avif">
    <source srcset="/img-800.webp 800w, /img-1200.webp 1200w" type="image/webp">
    <img src="/img-800.jpg" alt="Product photo" width="800" height="600" loading="lazy" decoding="async">
  </picture>
  ```
- [ ] **Font Subsetting**: Strip unused language glyphs and preload critical WOFF2 font files.

---

## 6. Rendering Architecture Tradeoffs

| Architecture | First Byte (TTFB) | First Content (FCP) | Interactive (TTI) | SEO | Best For |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CSR** (Client-Side) | Very Fast | Slow | Slow | Weak | Internal tools, dashboards, SaaS portals |
| **SSR** (Server-Side) | Moderate | Fast | Moderate | Excellent | E-commerce, news, social networks |
| **SSG** (Static Generation) | Fastest (CDN) | Fastest | Fast | Excellent | Blogs, documentation, marketing sites |
| **ISR** (Incremental) | Fastest (CDN) | Fastest | Fast | Excellent | Large e-commerce catalogs with millions of pages |
| **Streaming SSR** | Fast | Very Fast | Fast | Excellent | Complex dashboards with slow downstream APIs |

---

## 7. Diagnosing & Profiling Tools

- [ ] **Chrome DevTools Performance Panel**: Record user flows, identify Long Tasks ($> 50\text{ms}$ marked in red), analyze Main thread flame charts.
- [ ] **Lighthouse**: Automated audits for Performance, Accessibility, Best Practices, and SEO.
- [ ] **Web Vitals Chrome Extension**: Real-time overlay showing LCP, INP, CLS values.
- [ ] **PerformanceObserver API**: Measure metrics in production code:
  ```javascript
  new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntries()) {
      console.log('LCP Candidate:', entry.startTime, entry);
    }
  }).observe({ type: 'largest-contentful-paint', buffered: true });
  ```
