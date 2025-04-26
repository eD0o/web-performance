# 7 - Improving First Contentful Paint

As mentioned before, for FCP it doesn't matter what the content is, it could be anything visible on the screen.
There is an important relationship here: by `improving Time to First Byte (TTFB), First Contentful Paint (FCP) also improves`.
Additionally, `by further optimizing FCP, Largest Contentful Paint (LCP) will also benefit`.

Here’s a direct, clean resume of the transcript you posted:

---

## 7.1 - Removing Sequence Chains

When loading a page, `dependencies like CSS and fonts can create sequence chains that delay First Contentful Paint` (FCP).  
This happens when:

- An HTML file loads
- It references a CSS file
- That CSS uses @import to load another CSS
- That CSS references fonts, background images, etc.

Because CSS and fonts are render-blocking, `the browser waits for all of them before rendering anything`.

![](https://i.imgur.com/ZoHDxuy.png)

The same issue can happen with JavaScript, especially when:

- A script dynamically injects another script
- A module import (import) creates a new network request at runtime

---

### How to Remove These Chains:

✅ `Bundle your files at build time`, not at runtime.  
✅ `Use a module bundler` like:

- Webpack
- Rollup
- Vite

> For CSS specifically, you can use [Lightning CSS](https://lightningcss.dev/) — a `lightweight bundler that resolves imports and outputs a single CSS file`.

---

### Example:

Originally, a site had:

- style.css → imported base.css
- base.css → imported colors.css, typography.css, etc.
- Fonts requested only after parsing everything

✅ After bundling:

- `All styles combined into one styles.bundle.css`
- Browser loads it immediately without following import chains
- FCP happens much sooner

Fonts are still render-blocking, but the CSS chain is collapsed and faster.

---

## 7.2 - Preloading Resources

When loading a page, it's `important to start critical path resources as early as possible` to improve First Contentful Paint (FCP).

![](https://i.imgur.com/aLmy9SP.png)

A `common problem is with Google Fonts`:

- You typically insert a link to a CSS file from Google.
- That CSS then points to font files, which are only requested after the CSS is downloaded and parsed.
- This delays when fonts and text appear on the page.

### Current Optimization (by default):

Google Fonts suggests using:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
```

✅ Preconnect: Starts DNS lookup, TCP handshake, and TLS negotiation early.  
❌ `Does NOT fetch the actual font files early` — it just prepares the connection.

### How to Further Optimize:

✅ Use `<link rel="preload">` to fetch font files immediately.

Example:

```html
<link
  rel="preload"
  as="font"
  type="font/woff2"
  crossorigin
  href="/path-to-font-file.woff2"
/>
```

This:

- `Starts downloading fonts right away, even before CSS` arrives.
- Can significantly speed up FCP.

![](https://i.imgur.com/62ig6qV.png)

### Important Notes:

- CORS: Fonts and fetch requests need crossorigin attribute and proper CORS headers.
- Risk: Directly preloading Google's auto-generated font URLs is risky — filenames might change and break your site.
- Best practice:  
  👉 `Host fonts locally and preload them from your own server`.  
  👉 `Avoid relying on Google's CDN for critical fonts`.

### Benefits:

- Fonts start downloading immediately.
- Flattens the dependency chain.
- Reduces waiting time after CSS is loaded.
- Local hosting often yields faster and more reliable font loading compared to Google's hosted versions.

---
