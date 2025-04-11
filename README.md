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

> ❌ It's not up to you. You can’t just mark a tiny <div> at the top as "important" to cheat LCP. `Google analyzes the page visually`.

### What counts for LCP?

✅ Can be:

- <img>
- <video>
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

Preload key assets with <link rel="preload">

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

Claro! Aqui está a versão equivalente das suas anotações, mas agora focada em **CLS: Cumulative Layout Shift**:

---

## 2.2 - CLS: Cumulative Layout Shift

CLS measures `how much the visible content moves around unexpectedly` as the page loads.

> 💥 It quantifies visual _instability_ — those annoying jumps that happen when things load out of order.

### What causes layout shifts?

⛔ Unexpected layout changes caused by:

- Images without width/height
- Fonts loading late (FOUT/FOIT)
- Ads or embeds loading dynamically
- DOM injected late (e.g., banners, popups)
- Lazy-loaded content above existing content

✅ Expected shifts (user-initiated) don’t count:

- Click-triggered modal
- Expanding accordion on tap

### ⚙️ How is CLS Calculated?

Each layout shift has a score:

```text
CLS = impact fraction × distance fraction
```

- **Impact fraction** = % of the viewport affected
- **Distance fraction** = how far elements moved

CLS is the **sum of all shift scores** within a session window (up to 5s long, max 1s gap).

---

### 🛑 Examples of CLS

| Before Load                               | After Load                                |
| ----------------------------------------- | ----------------------------------------- |
| ![](https://i.imgur.com/7HZpGpT.png)      | ![](https://i.imgur.com/pP9THKM.png)      |
| 🟥 Text shifts due to late-loading banner | 🟥 CLS spike — layout jumped unexpectedly |

---

### 🎯 Best Practices to Avoid CLS

- ✅ Always define `width` and `height` for images and videos
- ✅ Use aspect-ratio boxes for media
- ✅ Preload fonts with `rel="preload"` to avoid FOUT/FOIT
- ✅ Reserve space for ads and embeds
- ✅ Avoid inserting DOM above existing content

> Tip: Don’t animate layout properties like `top` or `height` — use `transform: translate()` for smoother motion.

---

### 🧪 CLS in SPAs

- CLS can occur late — not just at initial load
- SPAs often inject dynamic content after route changes
- Use `layout stability techniques` consistently across routes

---

### 📊 How to Measure CLS

You can track CLS using the Performance API:

```js
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      console.log("Shift score:", entry.value, "Element:", entry.target);
    }
  }
}).observe({ type: "layout-shift", buffered: true });
```

Or: 

```js
import { getCLS } from "web-vitals";

getCLS((metric) => {
  console.log(metric);
});
```

Or use tools like:

- Lighthouse
- Web Vitals Chrome Extension
- PageSpeed Insights

---

### ✅ Good CLS Thresholds

| Score      | CLS Value  |
| ---------- | ---------- |
| Good       | ≤ 0.1      |
| Needs Work | 0.1 – 0.25 |
| Poor       | > 0.25     |

> CLS penalties increase if core content shifts more than 10% of the screen, especially if it happens late in the page lifecycle.

---

### 🧱 Common Fixes

| Problem                               | Solution                                       |
| ------------------------------------- | ---------------------------------------------- |
| No `width/height` on images           | ✅ Add dimensions or aspect-ratio              |
| Flash of invisible text (FOIT)        | ✅ Preload fonts, use font-display: swap       |
| Ads resizing after load               | ✅ Reserve fixed space, avoid collapsing gaps  |
| Injected banners/promo at top of page | ✅ Push them below fold or reserve space early |
