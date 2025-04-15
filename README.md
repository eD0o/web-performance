# 3 - Performance

Web performance measurement in JavaScript primarily relies on two browser APIs:

- Performance API
- PerformanceObserver API

These APIs provide granular `insights into timing, resource usage, and user interactions`.

---

## 3.1 - Performance API

The Performance API `gives access to high-resolution timestamps and performance metrics` for a web page.

### 🔹 performance.now()

`Returns a high-resolution timestamp` (in milliseconds) relative to performance.timeOrigin.

```js
const start = performance.now();
// ... some operation
const end = performance.now();
console.log(`Took ${end - start}ms`);
```

#### 🔸 Difference: `Date.now()` vs `performance.now()`

| Metric     | `Date.now()`        | `performance.now()`         |
| ---------- | ------------------- | --------------------------- |
| Based on   | Unix Epoch (1970)   | `performance.timeOrigin`    |
| Resolution | Milliseconds        | Fractional milliseconds     |
| Use case   | Logging, timestamps | Precise performance metrics |

![](https://i.imgur.com/7awHDos.png)

### 🔹 performance.timeOrigin

Represents the timestamp (similar to Date.now()) at which the `navigation or worker started`.

```js
const timestamp = performance.timeOrigin + performance.now();
```

This combination gives you a high-precision timestamp equivalent to Date.now().

---

### 🔹 performance.getEntries()

`Returns a list of all recorded performance entries`, including:

- Navigation
- Resource fetches (CSS, JS, images)
- Custom marks and measures

```js
const entries = performance.getEntries();
entries.forEach((entry) => console.log(entry.name, entry.startTime));
```

> This is `essentially what you see in the Network tab in DevTools`.

---

### 🔹 performance.mark(name)

`Creates a named timestamp (a "mark")` in the browser's performance timeline.

```js
performance.mark("start-heavy-task");
// ... run heavy operation
performance.mark("end-heavy-task");
```

---

### 🔹 performance.measure(name, startMark, endMark)

Measures the duration between two marks.

```js
performance.measure("task-duration", "start-heavy-task", "end-heavy-task");
```

You can then access it via `getEntriesByType('measure')`.

---

Absolutely! Here's a clearer and more structured version of your notes, following your same tone and format style:

---

## 3.2 - PerformanceObserver API

The PerformanceObserver `lets us passively collect performance metrics when the browser is idle`, without blocking or interfering with the main thread.

This is especially `useful when logging performance during runtime—so you don’t slow things down by measuring them`.

> 🧠 Ideal for tracking metrics like Core Web Vitals, long tasks, layout shifts, etc.

### Example: Observing Layout Shifts (CLS)

```js
const performanceObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`Layout shifted by ${entry.value}`);
  });
});

performanceObserver.observe({
  type: "layout-shift",
  buffered: true,
});

// type: "layout-shift" → we're observing shifts that impact Cumulative Layout Shift (CLS).

// buffered: true → ensures we also catch entries that happened before the observer was initialized.

// buffered: false (default): only observes new entries from this point forward.
```

### Other Observable Entry Types:

- "layout-shift" (CLS)
- "largest-contentful-paint" (LCP)
- "first-input" or "event" (INP)
- "resource" (images, scripts, CSS, etc.)
- "navigation" (full page loads)

### 🔍 Filtering by entry.entryType

Each PerformanceEntry has a entryType (like "resource", "layout-shift", etc). If you're observing multiple types or using getEntries(), you can filter like this:

```js
list.getEntries().forEach((entry) => {
  if (entry.entryType === "layout-shift") {
    console.log(`CLS shift: ${entry.value}`);
  }
});
```

Helpful when multiple entry types are observed and you want fine-grained control over handling them.

### 🧹 Managing Observers & Filtering

```js
performanceObserver.disconnect();
// ✅ Prevents memory leaks and keeps your app efficient — especially important in SPAs or long-lived sessions.
```

After you're done observing, always call disconnect() to stop the observer and free up memory.

Use it when:

- You're done collecting metrics
- The page/component unmounts
- You only need a one-time measurement

### Easy Core Web Vitals Tracking

You can also track Core Web Vitals with the [web-vitals](https://www.npmjs.com/package/web-vitals) library:

```js
import { onLCP, onCLS, onINP } from "web-vitals";

onLCP(console.log);
onCLS(console.log);
onINP(console.log);
```

This wraps PerformanceObserver under the hood and gives you simple callbacks for each metric.

---

## 3.3 - Browser Support

![](https://i.imgur.com/sLyUUWc.png)

![](https://i.imgur.com/BUmqbIp.png)

> 🧨 Safari still lacks full support for many Web Vitals APIs, which limits cross-browser consistency.

---
