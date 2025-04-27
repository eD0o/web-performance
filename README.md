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

> Modern browsers handle loading="lazy" natively; no extra JavaScript library is required ✅.

Browser behavior after lazy loading:

![](https://i.imgur.com/lOdnk40.png)

- Critical images like hero-mobile and hero-desktop are prioritized.
- Non-critical images wait until the browser has idle network time.
- They are discovered in the DOM, but download is deferred until the browser decides it's a good time.

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

Here’s a clean summary of what Todd Gardner explained:

---

## 8.2 - Eager Loading

If an image is important for LCP (like the hero image), we want the browser to start loading it earlier.  
Two main strategies:

| Strategy                                               | What it Does                                 | Browser Support            | Notes           |
| ------------------------------------------------------ | -------------------------------------------- | -------------------------- | --------------- |
| Preload (`<link rel="preload" as="image" href="...">`) | Tells the browser `early to fetch the image` | Works everywhere           | Most reliable   |
| `fetchpriority="high"` (in `<img>`)                    | `Signals this is important` to the browser   | Chrome, Edge (not Firefox) | Good extra hint |

> Important: `fetchpriority alone still waits` until parsing the image tag — `preload starts even earlier`.

### Example Code:

```html
<head>
  <link rel="preload" as="image" href="/path/to/hero.jpg" />
</head>

<body>
  <img src="/path/to/hero.jpg" fetchpriority="high" alt="Hero Image" />
</body>
```

### Key Points:

- Preload allows the browser to start fetching the resource earlier, even before the HTML parser reaches the img element.
- fetchpriority="high" makes sure the image stays top-priority during the loading queue.
- `You can and should use both together for your LCP images`.
- No need for crossorigin for same-origin images in this context.
- fetchpriority is optional but recommended where supported (especially for SEO-critical pages).

---
