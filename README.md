# 2 - Core Web Vitals

Core Web Vitals are `metrics that help measure real-world user experience for performance`. They focus on:

1. How fast your site visibly loads
2. How smooth the experience is while it loads
3. How quickly users can interact

## Main Metrics:

- LCP: Largest Contentful Paint
- CLS: Cumulative Layout Shift
- INP: Interaction to Next Paint

> 💡 Google uses these metrics in its search ranking algorithms.

---

## 2.1 - LCP: Largest Contentful Paint

LCP measures `how fast the most important visible element on the page fully loads`.  
It stops measuring once the user interacts with the page.

But... who decides what's the most important content?

> ❌ It's not up to you. You can’t just mark a tiny div at the top as "important" to cheat LCP. `Google analyzes the page visually`.

### What counts for LCP?

✅ Can be:

- img
- video
- CSS background-image
- Block-level text elements

⛔ Won’t count if:

- opacity: 0
- display: none
- Size < 100%
- Low entropy images (e.g. blurred placeholders)

🛑 LCP calculation stops after:
First user interaction (e.g., click, tap, keypress)
`Hovering does NOT count`.

🧪 LCP in SPAs
DOMContentLoaded/load events exist but are less meaningful in SPAs

`LCP is better for measuring perceived performance in client-rendered apps`

🎯 Best Practices
Optimize hero image (usually the LCP element)

Use proper image formats (e.g., WebP, AVIF)

Lazy-load below-the-fold assets

Avoid overly large banners unless intentionally the LCP

Preload key assets with `<link rel="preload">`

---

### 2.1.1 - Entropy

Entropy helps Google detect whether an image is `"visually meaningful"` enough to count for LCP.

`Bits per visible pixel`

#### High Entropy (Valid LCP)

- Original: 3.9 MB → 31 million bits
- Rendered at 2800x1200 → 3.3 million pixels
- Entropy: 9.39 bits/pixel  
  ✅ This qualifies as LCP.

![](https://i.imgur.com/AIZIoWf.png)

#### Low Entropy (Ignored by LCP)

- Placeholder image: 17 bytes
- Rendered at 200x88
- Entropy: 0.001  
  ⛔ Too little visual info — ignored by Google.

![](https://i.imgur.com/kmsidjH.png)

> Even if it makes UX feel faster, Google ignores it for LCP.

#### 🖼️ Lazy-loaded Images and LCP

Do lazy-loaded images count for LCP?  
✅ Yes — but only once they’ve fully loaded.

`LCP measures when the largest visible content is fully rendered`. If that content is a lazy-loaded image:

- `LCP waits for it to load — if it's visible`.
- If it’s not in the viewport when measuring, it’s ignored.

---

#### ⚠️ Common Issues with Lazy-loaded Images

| Scenario                                  | Effect on LCP       |
| ----------------------------------------- | ------------------- |
| loading="lazy" on LCP image               | ❌ Delays LCP event |
| Low-entropy placeholder image             | ❌ Ignored by LCP   |
| Lazy-loaded image not in initial viewport | ❌ Ignored          |

---

#### ✅ Best Practices

- ❌ `Avoid loading="lazy" on your hero` or key visual.
- ✅ `Use <link rel="preload" as="image" href="..."> for important images`.
- ✅ Ensure images have `sufficient entropy to qualify for LCP` (bigger than 0.05 bits per pixel).

---

### 📏 How to Measure Entropy

You can calculate image entropy using the browser console:

```js
console.table(
  [...document.images].map((img) => {
    const entry = performance.getEntriesByName(img.currentSrc)[0];
    const bytes = entry?.encodedBodySize * 8;
    const pixels = img.width * img.height;
    return { src: img.currentSrc, bytes, pixels, entropy: bytes / pixels };
  })
);
```

---

### ⏱️ LCP Load Sequence Example

![](https://i.imgur.com/RnFxGZ8.png)

---

### ✅ Good LCP Thresholds

![](https://i.imgur.com/R8XL8tz.png)

| Score      | LCP Time  |
| ---------- | --------- |
| Good       | ≤ 2.5s    |
| Needs Work | 2.5s – 4s |
| Poor       | > 4s      |

⚠️ What happens if you're over the LCP threshold?
2.5s is not arbitrary – based on behavioral studies: users feel interrupted after 2s

`LCP > 2.5s leads to ranking penalties`, though the exact formula is proprietary.

---

## 2.2 - CLS: Cumulative Layout Shift

CLS (Cumulative Layout Shift) `measures visual stability, capturing how often and how drastically elements move unexpectedly` on screen. It `focuses on the user’s viewport` and is a key Core Web Vital.

---

### 📦 What Causes Layout Shifts?

- New elements (like promo banners) pushing content down.
- Images, fonts, or iframes `loading after the initial paint`, shifting elements.
- Users interacting (scrolling, clicking) while the layout is still being adjusted.

> `Only shifts that occur in the user’s viewport count`. If the element was visible and moved, it contributes to CLS.

It's possible to see it happening here: https://shifty.site/

### How to Calculate Layout Shift

Layout Shift is measured by multiplying two factors:

**Impact Fraction × Distance Fraction = Layout Shift Value**

This value is calculated for each unexpected shift, and `all values are summed up to get the final score` for Cumulative Layout Shift (CLS).

![](https://i.imgur.com/ZxhwJ4C.png)

> ⚠️ Shifts caused by user interactions and those that happen within 500ms of input are excluded from CLS.

---

### ✅ What Doesn’t Count?

- Elements that don’t move during load (e.g., fixed/stable headers).
- Content inside a canvas — CLS only considers the element’s bounding box.
- Shifts outside the viewport, unless they eventually move content into view.
- Properly reserved space via CSS or HTML (width, height, aspect-ratio).

> A layout shift is measured only when something visually moves unexpectedly.

---

### 🔁 CLS Is Not a Fixed Score

CLS is `not deterministic — users experience different layout behaviors` due to:

- Device and screen size
- Network conditions
- Scroll timing and interaction

> Google aggregates thousands of layout shift instances and `reports the 75th percentile CLS across real-user data`.

---

### 🎯 CLS in Action

| 🧪 Example Scenario                                       | Does it count toward CLS? |
| --------------------------------------------------------- | ------------------------- |
| Header stays in place during load                         | ❌ No                     |
| Promo banner pushes content down                          | ✅ Yes                    |
| Image with unknown size loads in later                    | ✅ Yes                    |
| Skeleton placeholder is swapped with content of same size | ❌ No                     |
| Canvas content moves inside but canvas size is static     | ❌ No                     |
| User scrolls while layout is still adjusting              | ✅ Yes                    |

---

### 🧱 Best Practices to Prevent CLS

| 🛠️ Technique                              | Prevents CLS? |
| ----------------------------------------- | ------------- |
| Set image width and height attrs          | ✅ Yes        |
| Use CSS aspect-ratio for elements         | ✅ Yes        |
| Lazy-load images with dimensions          | ✅ Yes        |
| Use skeleton loaders or placeholders      | ✅ Yes        |
| Load fonts with font-display: optional    | ✅ Yes        |
| Animate size/position without reservation | ❌ No         |

---

### 📱 Responsiveness and CLS

Responsive design `can unintentionally trigger layout shifts`, especially when:

- Styles adapt via media queries without reserving layout space
- Elements reposition or resize due to viewport changes (e.g., mobile breakpoints)
- Components re-render differently at different screen sizes without layout stabilization

> 💡 Pro tip: `Always test CLS on multiple screen sizes and devices to ensure responsive layouts are stable`.

Strategies to handle responsivity without shifting:

- Use `min-height and aspect-ratio to reserve layout slots` before media queries take effect.
- Avoid injecting layout-altering DOM changes based on screen width after the initial render.
- `Avoid flex/grid reflows by using visibility or opacity` to toggle UI instead of inserting/removing DOM nodes.

---

### 🧠 Summary

- CLS is about visual stability — what moves, when, and where.
- It’s user-centric: only what’s visible and changes matters.
- CLS is measured across real sessions, not just a single run.
- You can’t just fix it “once” — it requires layout planning across different viewports, content types, and loading scenarios.

---

## 2.3 - Flame Chart: Visualizing Browser Tasks

A flame chart `is a time-based visualization of how the browser’s main thread executes tasks`. It’s commonly used in Chrome DevTools (Performance tab) and `shows a stacked timeline of browser activities` in milliseconds or microseconds.

---

### 📊 How It Works

Each horizontal bar in a flame chart represents a task or function execution:

- Bars are stacked to show parent/child relationships.
- The width of each bar shows how long that task took.
- The vertical stack shows call depth: a function calling another function, which calls another, and so on.

```js
function task1() {
  task2();
}

function task2() {
  task3();
}

function task3() {
  // Do something expensive
}
```

This call stack would appear as:

![](https://i.imgur.com/5DYPq5m.png)

---

### 🎨 Color Coding in Flame Charts (Chrome)

| Color        | Meaning                                      |
| ------------ | -------------------------------------------- |
| Gray         | Top-level browser task                       |
| Blue         | Parsing HTML                                 |
| Pink         | Layout and paint                             |
| Dark yellow  | JavaScript setup (evaluate, compile, events) |
| Light yellow | Active JavaScript execution                  |
| Green        | Extensions (usually not relevant)            |

---

### 🧠 Why Flame Charts Matter

The main thread is a shared resource — all of this happens in one thread:

- JavaScript execution
- Layout and rendering
- Handling user interactions
- Painting the screen

> 💡 If your JavaScript takes too long, it can block layout, user input, or rendering. Flame charts help you find what’s taking too long and where the bottlenecks are.

---

Another Example:

```html
<html>
  <body>
    <script>
      window.addEventListener("load", () => {
        var el = document.createElement("div");
        el.innerHTML = "<h1>Hey</h1>";
        document.body.appendChild(el);
      });
    </script>
  </body>
</html>
```

1. HTML parsed → triggers a top-level task (gray)
2. Script found → compiled and executed (dark yellow)
3. addEventListener("load") is registered → function is not yet executed
4. On load, a new task is triggered (gray)
5. Function is executed: creates element, sets innerHTML, appends (light yellow)
6. appendChild triggers layout and paint → possible CLS (pink)

---

## 2.4 - INP: Interaction to Next Paint

How quickly users can interact.

### What’s an interaction?

- Click
- Drag
- Touch
- Keypress

> Exception: Scroll.

`Every time the user interacts`, it generates an INP. `The worst interaction time will be the score`, and the worst time cannot be determined until the interaction has completed. `If no interaction occurs, there's no INP score`.

![](https://i.imgur.com/2EHt4TC.png)

INP `measures the latency between the user’s interaction and the time the browser can visually respond`. A high INP means the user may experience delays in seeing feedback or action on their input, leading to a poor perception of site performance.

INP is `heavily influenced by device capabilities`, meaning slower or lower-powered devices might experience higher latency.

> A good INP score is under 200ms.

### Why INP matters?

- User experience: A `high INP can make your site feel sluggish`. Fast interactions result in better user engagement and satisfaction.
- Mobile impact: Since mobile devices often have limited processing power, a good `INP score becomes more crucial for mobile performance`.

### How to track INP?

- Chrome DevTools: View INP metrics using the Web Vitals panel.
- Web Vitals Extension: Use it for easy monitoring of INP on your page.

## 2.5 - Time to First Byte (TTFB)

TTFB is the time from when the browser sends a request to the server, until it receives the first byte of the response. `How quickly your host responds`.

It's `not a CWV, but still an important metric for server-side performance`. Often used in web performance audits.

![](https://i.imgur.com/gmFF8V4.png)

![](https://i.imgur.com/iXEHR4N.png)

### What affects TTFB?

- Server response time
- Network latency
- DNS resolution
- CDN caching

TTFB `is a backend performance metric`, and one of the first indicators of how fast your server begins to respond.

> ⏱ A good TTFB is under 800ms, but for optimal LCP performance, aim for under 200ms if possible.

`TTFB affects your LCP. If your TTFB is bad, even with optimized images and CSS, your LCP can never be faster than that initial wait.`

That’s why reducing TTFB (via CDN, server optimization, or caching) is often one of the first steps in improving LCP.

---

## 2.6 - Legacy Metrics

### 2.6.1 - First Input Delay (FID) [Legacy CWV]

FID measures the time between the user's first interaction with the page (like clicking a link or tapping a button) and the `time the browser is actually able to respond to that interaction`.

> ✅ It reflects the responsiveness of your site during page load.

#### Why FID matters?

- `A long delay means the main thread is blocked` (often by heavy JavaScript), so the browser can’t respond quickly.
- A good FID is less than 100ms.

> 🔁 `Replaced by INP` (Interaction to Next Paint) as a Core Web Vital in March 2024. Still useful to understand input responsiveness, but not used for CWV scoring anymore.

---

### 2.6.2 - First Contentful Paint (FCP) [Legacy CWV]

FCP `measures how long it takes from the moment the page starts loading to when any part of the content is rendered` (text, image, SVG).

Still measured, but not one of the main 3 Core Web Vitals anymore (those are now LCP, INP, CLS). FCP is `often used to complement LCP insights`.

![](https://i.imgur.com/qeAsv1T.png)

#### What’s considered “content”?

- Text
- Images (including background)
- SVGs
- Canvas

> 🧩 It’s the moment `something meaningful appears on screen`, but not necessarily what the user is waiting for.

- A good FCP is under 1.8 seconds.

`FCP helps you understand how quickly users perceive that the page is loading.`
