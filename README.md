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
