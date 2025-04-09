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

## 2.1.1 - Entropy

Entropy helps Google detect whether an image is `"visually meaningful"` enough to count for LCP.

`Bits per visible pixel`

### High Entropy (Valid LCP)

- Original: 3.9 MB → 31 million bits
- Rendered at 2800x1200 → 3.3 million pixels
- Entropy: 9.39 bits/pixel  
  ✅ This qualifies as LCP.

![](https://i.imgur.com/AIZIoWf.png)

### Low Entropy (Ignored by LCP)

- Placeholder image: 17 bytes
- Rendered at 200x88
- Entropy: 0.001  
  ⛔ Too little visual info — ignored by Google.

![](https://i.imgur.com/kmsidjH.png)

> Even if it makes UX feel faster, Google ignores it for LCP.

### 🖼️ Lazy-loaded Images and LCP

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
