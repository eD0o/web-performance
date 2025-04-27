# 8 - Improving Largest Contentful Paint (LCP)

Goal: Make the `largest visible content load as early and quickly as possible`.

Why we talked about TTFB and FCP first ?

- TTFB (Time to First Byte) and FCP (First Contentful Paint) are building blocks.
- `By improving TTFB and FCP, you automatically help LCP` because the critical resources (like images) can start loading sooner.

| Part              | What it means                                                                                | Focus                                                    |
| :---------------- | :------------------------------------------------------------------------------------------- | :------------------------------------------------------- |
| Resource Delay    | Time waiting before starting to download the LCP resource                                    | Mostly fixed by improving TTFB and FCP                   |
| Resource Duration | Time it takes to download the LCP resource                                                   | Main focus for optimization                              |
| Render Delay      | Time after the resource is ready but the browser hasn't yet rendered it (due to JS blocking) | Rarely a problem unless your site has massive JavaScript |

---

## 8.1 - Lazy Loading

Lazy loading is about `removing non-critical resources from the critical rendering path` to prioritize loading of the LCP element.

- Context:  
  After optimizing fonts and CSS, a lot of non-critical images were still being downloaded early, competing with the LCP image.

- Strategy:

  - `Lazy load all images and iframes except the LCP candidate`.
  - `Teach the browser that certain images are lower priority`.
  - This reduces resource delay for the important LCP asset.

- How to implement:

  - Bulk `replace all <img> tags by adding loading="lazy"` attribute.
    - Search across the entire project.
    - Apply to HTML and dynamically rendered images in JavaScript.
  - Exception:  
    The `LCP image(s) must NOT be lazy loaded`.
    - In this case:
      - hero-desktop
      - hero-mobile
    - Remove any loading="lazy" from these images to ensure they download immediately.

> Today, just adding loading="lazy" is enough — no library needed ✅.

Browser behavior after lazy loading:

![](https://i.imgur.com/lOdnk40.png)

- Critical images like hero-mobile and hero-desktop are prioritized.
- Non-critical images wait until the browser has idle network time.
- Lazy-loaded images might still be discovered early but won't start downloading immediately.

- Above vs Below the Fold:
  - Below-the-fold images: Always lazy load.
  - Above-the-fold images:
    - Lazy load if they are not essential for initial user experience.
    - Keep non-lazy only the most important visual elements (like LCP).

### Quick Visual

| Image Type               | Strategy             |
| ------------------------ | -------------------- |
| LCP candidate            | No `loading="lazy"`  |
| Above-the-fold (non-LCP) | Possibly lazy load   |
| Below-the-fold           | Definitely lazy load |

> \*fold -> bottom edge of the visible part of the page before scrolling.

---
