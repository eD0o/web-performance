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