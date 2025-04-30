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
