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

## 8.3 - Image Formats

Even if we load an image (like the LCP image) as early as possible, it might still take a long time to display. To improve this, `we must send fewer bytes — reduce the image size as much as possible`.

HTTP compression (e.g., gzip, Brotli) works great for text, but not for images because images are already compressed formats.

> Running an image through text compression saves very little, sometimes almost nothing.

### Choosing the Right Image Format

| Format | File Size |
| :----- | :-------- |
| JPG    | 13 KB     |
| PNG    | 5.5 KB    |
| WebP   | 2.7 KB    |
| AVIF   | 2.6 KB    |

- Visually, the images look identical, but `file sizes vary drastically`.
- `AVIF and WebP are much smaller compared to JPG and PNG`.
- `WebP is broadly supported`; AVIF can sometimes be much slower to decode on older devices, but has growing support.

> Using true SVGs are a great idea, mainly for icons and tiny images.

![](https://i.imgur.com/KfhFcrJ.png)

### Format Recommendations

- Photos: JPG used to be the best, but now WebP and AVIF perform better.
- Illustrations (graphics, logos): PNG was common, but again, WebP and AVIF are better options now.
- Between WebP and AVIF, the size difference is small — using either is a big win compared to older formats.

### When You Can't Use WebP or AVIF

- Sometimes you can't use modern formats (due to `browser support or system constraints`).
- Tools like `TinyPNG (https://tinypng.com) can optimize existing PNGs and JPGs dramatically`.
- Example: a PNG compressed from 57 KB down to 15 KB, with no visible quality loss.

![](https://i.imgur.com/DrYjNQs.png)

---

## 8.4 - Responsive Images

- `Not every device needs the largest version of an image`.
- High-resolution displays (e.g., Retina screens) might need a 2800px wide image.
- Mobile devices might only need 720px, 600px, or even 300px wide versions.
- Serving appropriately sized images saves a lot of bytes and improves loading speed.

Example:

```html
<picture class="illustration">
  <!-- mobile -->
  <source
    media="(max-width: 720px)"
    srcset="
      /hero-mobile.png?width=360   360w,
      /hero-mobile.png?width=720   720w,
      /hero-mobile.png?width=1440 1440w
    "
  />
  <!-- desktop -->
  <source
    media="(min-width: 721px)"
    srcset="
      /hero-desktop.png?width=720   720w,
      /hero-desktop.png?width=1440 1440w,
      /hero-desktop.png?width=2800 2800w
    "
  />
  <!-- default -->
  <img
    src="/hero-desktop.png?width=2800"
    alt="Developer Stickers Online"
    fetchpriority="high"
    height="1200"
    width="2800"
  />
</picture>
```

- Each source:

  - `Defines a condition with the media attribute` (e.g., screen width).
  - `Provides alternative image options` using the srcset attribute.

- The img at the end:
  - Is mandatory.
  - `Ensures an image is always displayed`.
  - Acts as a fallback `if none of the source conditions` match.

## 8.5 - Optimizing Images

In the 00_setup folder, there are some tools that can help with image optimization:

![](https://i.imgur.com/0bAjeKA.png)

### 8.5.1 - Image Resizing (imagePngResizer.mjs)

- Goal: `Create smaller versions of each image` to ship less data.
- Process:
  - Use a tool (Jimp) to create resized copies of all PNG images.
  - Sizes generated: 360px, 720px, 1024px, 1400px, 2800px width.
- Command:
  ```bash
  npm run image-png-resizer
  ```
- Outcome:  
  For each original image, multiple resized versions are now available under public/assets/image/R/.

#### Example:

| Size     | File Size |
| -------- | --------- |
| Original | 1.5 MB    |
| 720px    | 800 KB    |
| 360px    | 250 KB    |

### 8.5.2 - PNG Optimization (imagePngOptimize.mjs)

- Goal: Further `reduce file size without visible quality loss`.
- Tool: [imagemin](https://github.com/imagemin/imagemin)
- Process:
  - Optimize all resized PNGs and output into a min/ directory.
- Command:
  ```bash
  npm run image-png-optimize
  ```
- Results:
  - Example:
    - Before optimization: 1.5 MB
    - After optimization: 470 KB
  - Visual Quality: No noticeable difference.

### 8.5.3 - Convert PNGs to WebP (imagePngToWebP.mjs)

- Goal: Create `even smaller versions by using WebP format`.
- Tool: imagemin-webp
- Process:
  - Convert optimized PNGs (from min/) into WebP files.
  - Output into a webP/ directory.
- Command:
  ```bash
  npm run image-png-to-webp
  ```
- Results:
  - Example:
    - `After WebP conversion: 69 KB`
      > Compared to original: from 1.5 MB → 69 KB
  - Visual Quality: No visible difference.

### 8.5.4 - Update Image References

- Goal: `Make the site load WebP images instead of PNGs`.
- Steps:
  - Use regex search/replace in VS Code:
    - Find: /assets/image/(.\*)\.png
    - Replace with: /assets/image/webP/$1.webp
- Effect:  
  All references in HTML and JS now point to lighter WebP versions.

### 8.5.5 - Implement Responsive Images

- Goal: `Load different image sizes depending on screen size` using the picture element.

- Sample structure:

  ```html
  <picture class="illustration">
    <source
      srcset="
        /assets/image/webP/hero-desktop-360.webp   360w,
        /assets/image/webP/hero-desktop-720.webp   720w,
        /assets/image/webP/hero-desktop-1400.webp 1400w
      "
      sizes="(max-width: 720px) 360px, (max-width: 1400px) 720px, 1400px"
      type="image/webp"
      fetchpriority="high"
    />
    <img
      src="/assets/image/webP/hero-desktop-1400.webp"
      alt="Hero Image"
      width="1400"
      height="auto"
    />
  </picture>
  ```

- Benefits:
  - `Browser only downloads the size it needs`.
  - High fetch priority improves LCP without over-fetching.

### 8.5.6 - Remove Preloads for LCP Image

- Why?  
  Preloading both desktop and mobile hero images was `no longer effective in this case because the exact needed image is not known at preload time (depends on screen width)`.
- Action:  
  Remove link rel="preload" for hero images.
- Alternative:  
  Rely on fetchpriority="high" inside picture, nothing that it’s not yet supported by Firefox.

### 8.5.7 - Final Results

- Page Load Check:
  - Site renders correctly with all images replaced by responsive WebPs.
- Performance Improvement:
  - LCP dropped dramatically to 454 milliseconds.
  - Page load time much faster (initial metric reported: ~5 seconds).
  - Huge bandwidth savings: from 1.5MB → ~70KB per key image.

### 📋 Key Takeaways

✅ Resize images at multiple resolutions.  
✅ Optimize images (lossless compression).  
✅ Convert to modern formats (WebP).  
✅ Update code references to point to new assets.  
✅ Use picture + srcset for responsive images.  
✅ Remove ineffective preloads if necessary.

---

## 8.6 - Caching

Caching is a `fundamental web performance technique that reduces redundant network requests`, speeds up load times for return visits, and lowers bandwidth usage.

### 8.6.1 - Server-Side Caching (via CDN)

- What it is: `Storing content on servers geographically closer to users to avoid hitting the origin server repeatedly`.
- How it works: A `CDN caches static files` (e.g., hero-desktop.png) and serves them directly.
- Result: Faster access due to reduced latency and lower server load.

### 8.6.2 - Browser-Side Caching

#### a. Validation-Based Caching

- The server returns:
  - ETag: A unique hash of the file.
  - Last-Modified: Timestamp of last change.
- On repeat requests, the browser sends:
  - If-None-Match: with the ETag.
  - If-Modified-Since: with the last modified date.
- `If the file hasn’t changed, the server returns 304 Not Modified (no body) – resulting in smaller payloads` but still involving a request.

#### b. Expiration-Based Caching

- The server returns:
  - Cache-Control: max-age=7200 (e.g., 2 hours).
  - Expires: future-date (optional).
- `The browser reuses the file without making any request until the expiration time is reached`.
- Zero request overhead = maximum speed.

> ⚠️ Be careful when caching files like scripts.js for long periods. If the content changes, browsers won’t fetch the new version unless the filename changes.

### 8.6.3 - The Cache Invalidation Problem

- Long-term caching can backfire if a file changes but retains the same name.
- Best practice:
  - `Use hashed filenames` (e.g., scripts.8f9a3b.js) to "bust" the cache.
  - Most bundlers (Webpack, Vite, etc.) automate this.
- `Ensures users always get the latest version without disabling aggressive caching`.

### 8.6.4 - Enabling Caching Headers

- In your app’s config (e.g., performance.config), you can enable caching headers:
  - ✅ `Enable 304 caching headers`: returns ETag and Last-Modified for static assets.
  - ✅ `Enable Cache-Control`: to set how long the browser should retain the file.

Example: Behavior:

- First request:

  - Browser asks for hero-mobile-1400.png.
  - Server returns the file along with:
    - ETag
    - Last-Modified
    - Cache-Control: max-age=7200

- Return visit:
  - `If cache is valid, the browser may not even make a network request`.
  - Chrome `may return the asset from memory cache (0ms load time, no request logged)`.
  - If the request is made and the file hasn’t changed, the server returns:
    - 304 Not Modified

Important:

- This significantly improves performance for return users.
- First-time visitors will still have to load everything fresh, `but caching pays off on all subsequent visits`.
