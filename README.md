# 9 - Improving CLS & INP

## 9.1 - Improving CLS

### 9.1.1 - What Causes CLS?

CLS `occurs when elements on the page shift unexpectedly`. Common causes:

- Images or iframes without defined dimensions.
- Web fonts causing text reflows.
- Ads or banners loading late.
- Injected content pushing other content.
- Animations that change layout.

### 9.1.2 - Main Strategy: Reserve Space Early

To prevent layout shifts, the `page should reserve the space of dynamic content before it fully loads`.

#### ✅ Best Practices:

- Use aspect-ratio  
  `Define the natural width/height ratio` to maintain a box’s shape:

  ```css
  .image-wrapper {
    aspect-ratio: 16 / 9;
  }
  ```

  This works great for images, iframes, or videos and is responsive by default.

- Define width and height or use CSS classes with fixed `min-height / min-width`  
  Especially useful for non-media containers like headers or banners:

  ```css
  .header-wrapper {
    min-height: 100px;
  }
  ```

- `Use intrinsic sizing` for responsive behavior  
  Combine aspect-ratio with max-width: 100% and height: auto:
  ```css
  img {
    aspect-ratio: 4 / 3;
    max-width: 100%;
    height: auto;
  }
  ```

### 9.1.3 - Additional CLS Strategies

#### Preload Fonts

- Avoid layout shift due to font swapping.
- Use:
  ```html
  <link
    rel="preload"
    href="font.woff2"
    as="font"
    type="font/woff2"
    crossorigin
  />
  ```
- In CSS:
  ```css
  @font-face {
    font-family: "CustomFont";
    src: url("font.woff2") format("woff2");
    font-display: swap; /* or optional */
  }
  ```

#### Avoid Injecting Content Above Existing Layout

- Do not push down layout with banners or toolbars.
- If dynamic:
  - Reserve space using min-height.
  - Or make it overlay with position: absolute / fixed.

#### Use Skeleton Loaders or Placeholders

- Placeholder boxes can reserve space visually:
  ```html
  <div class="card skeleton"></div>
  ```
- Blur-up or LQIP (Low-Quality Image Placeholder) for images.

#### Avoid Layout-affecting Animations

- Animate only transform or opacity, not dimensions:
  ```css
  transition: transform 0.3s ease;
  ```

#### Reserve Space for Ads, Embeds, and Widgets

- Allocate fixed height for ad slots or iframes.
- Never collapse container if content is missing — use fallback space.

### ✅ Summary

| Strategy                     | Prevents CLS?                       | Responsive?                               |
| ---------------------------- | ----------------------------------- | ----------------------------------------- |
| aspect-ratio                 | ✅                                  | ✅                                        |
| width + height (px absolute) | ✅                                  | ❌ (`unless adjusted with media queries`) |
| width + height (vw/vh)       | ✅                                  | ✅ (viewport-relative)                    |
| width + height (%-based)     | ⚠️ Only if parent has explicit size | ✅                                        |
| min-height / placeholder     | ✅                                  | ✅                                        |
| Font preload + font-display  | ✅                                  | ✅                                        |
| Avoid top injection          | ✅                                  | ✅                                        |
| Animate transform/opacity    | ✅                                  | ✅                                        |

---

Absolutely! Here's your revised notes with integrated corrections, clarity improvements, and the new JavaScript examples:

---

Your notes are already well-structured and technically solid! Here's a reviewed and slightly refined version for clarity, precision, and flow:

---

## 9.2 - Improving INP (Interaction to Next Paint)

### Demonstration of INP Measurement

INP (Interaction to Next Paint) reflects the worst interaction latency experienced by a user on a page.

To demonstrate how INP works, it must be measured in a real user interaction scenario—for example, clicking the Add to Cart button.  
`Unlike metrics like LCP or FCP, which are captured passively during page load, INP requires an actual interaction to be triggered`.

> Even if 9 out of 10 interactions are fast, `a single slow interaction defines your INP`.  
> That's why `reviewing all user interaction triggers on the page is essential`. It only takes one poorly optimized interaction (e.g., a slow dropdown, toggle, or button) to degrade the score.

#### Example Measurement Flow

1. Triggering Interaction: Simulate a user action like clicking "Add to Cart". `If no immediate feedback is shown, the user perceives lag`.
2. Identifying Bottlenecks: With tools like Chrome DevTools, you may notice a delay between the user click and UI update.

For example, if the click handler runs analytics, fetches data, or processes logic before updating the UI, this creates delays.  
A 1200ms gap before any visual feedback is a strong sign of poor INP.

### Key Concept: Yielding the Main Thread

To optimize INP, the key idea is to yield control back to the browser, allowing it to `paint the UI before executing heavy JavaScript logic`.

A common "bad" sequence when handling a click:

- Capturing the event
- Running validation or analytics
- Performing a fetch or mutation
- Then finally updating the UI

> When these operations run synchronously, the browser is blocked from rendering visual feedback, causing interaction delays.

---

### Solution: Yielding to the Main Thread

To improve responsiveness, we can break up the task so that rendering happens before heavy processing. Two reliable strategies:

- requestAnimationFrame: `Allows the browser to schedule a paint before continuing`.
- setTimeout: `Defers logic to the next macrotask`, letting the browser breathe.

---

### ✅ Code Refactor for Improved INP

```js
async function handleAddToCart(event) {
  const productId = getProductId(event);

  // Yield to main thread for UI update
  requestAnimationFrame(() => {
    updateButtonUI(); // Feedback like "Added"

    // After painting, update analytics and perform other tasks
    setTimeout(() => {
      updateAnalytics(productId);
      addToCart(productId);
    }, 0);
  });
}
```

- Flame Chart: After refactoring, DevTools will show the interaction being split into smaller blocks. This frees up the main thread for quicker paint.
- INP Measurement: The delay between interaction and visual feedback is reduced, improving the INP score.

### 🧪 More Examples for Optimizing INP

#### 🔁 Example 1: Prioritizing UI Feedback Before Logic

```js
button.addEventListener("click", () => {
  // Immediate visual feedback
  button.textContent = "Processing...";

  requestAnimationFrame(() => {
    setTimeout(() => {
      doHeavyProcessing();
      button.textContent = "Done!";
    }, 0);
  });
});
```

#### 🔁 Example 2: Awaiting Paint Before Running Heavy Logic

```js
button.onclick = async () => {
  showFeedback(); // e.g., spinner or text change

  // Let browser render before continuing
  await new Promise(requestAnimationFrame);

  await doHeavyWork(); // Proceed with data fetch or business logic
};
```

#### 🔁 Example 3: Using `queueMicrotask` for Lightweight Deferral

```js
button.addEventListener("click", () => {
  updateButtonUI(); // Immediate user feedback

  // Run light logic soon after current execution
  queueMicrotask(() => {
    runSynchronousLogic(); // E.g., simple validation
  });
});
```

#### 🔁 Example 4: Using `scheduler.postTask` (Experimental API)

```js
button.addEventListener("click", () => {
  showVisualFeedback(); // e.g., loader icon

  scheduler.postTask(
    () => {
      runHeavyTask(); // Runs in background without blocking UI
    },
    { priority: "background" }
  );
});
```

---
