# 6 - Improving Time to First Byte

As said before, TTFB measures `how quickly a server responds to a request` and is a foundational part of FCP and LCP. `Improving TTFB enhances downstream metrics like FCP and LCP, as it shortens the delay before the browser starts rendering.`.

## 6.1 - TTFB Baseline Example

- Tested from Brazil using Chrome DevTools with throttled network (Coffee WiFi: 10Mbps down, 4Mbps up, 50ms latency).
- Page load analyzed in Chrome Performance tab.

![](https://i.imgur.com/RVfYvp2.png)

Results:

![](https://i.imgur.com/0JJQNvq.png)

- Total load took ~7 seconds.

## 6.2 - Gzip & Brotli Compression

Reduce the number of bytes sent over the network by `compressing plain text assets (HTML, CSS, JS), which improves TTFB and overall load time`.

### 🧰 Tactics Covered

- Enable HTTP compression via Gzip and Brotli (Next.js uses Brotli by default)
- `Understand how the browser negotiates compression with the server`
- Observe actual network savings via DevTools

### 📦 Compression Algorithms

| Type    | Compression Ratio                          | Notes                              |
| ------- | ------------------------------------------ | ---------------------------------- |
| Gzip    | ~25% of original                           | Supported widely, great baseline   |
| Brotli  | ~15–30% of original                        | Best compression, slower than Gzip |
| Deflate | Middle ground                              | Older, less common today           |
| Zstd    | Mentioned via headers but not demonstrated |

### 🧪 Example Results (real test)

- HTML file:
  - Uncompressed: 33 KB
  - Gzip: 6 KB
  - Brotli: Slightly larger than Gzip (in this case)

> Brotli has a higher compression ratio but can have longer compression time, and for small files, Gzip may achieve similar or even better results due to faster compression overhead.

### 🔧 Server-Side Setup

- Server `configured via performance-config.js`
- Enabled compression for both Gzip and Brotli
- Used Node/Express to instruct static file serving with compression

```js
// performance-config.js
module.exports = {
  /**
   * Whether to use basic GZip Compression on response bodies, when supported by
   * the requesting browser.
   * @see https://developer.mozilla.org/en-US/docs/Glossary/gzip_compression
   */
  enableGzipCompression: false,

  /**
   * Whether to use the Brotli Compression on response bodies, when supported
   * by the requesting browser.
   * @see https://developer.mozilla.org/en-US/docs/Glossary/Brotli_compression
   */
  enableBrotliCompression: false,
};
```

```js
// app.js
const express = require("express");
const { resolve } = require("path");

// This uses http-compression, a third-party package for simplified testing. In production, you'd typically use compression (Express middleware) or server-level compression (e.g., Nginx).
const compression = require("http-compression");

const api = require("./routes/api");
const performanceConfig = require("../performance-config");

const app = express();

/**
 * Simulating real-world delays for server processing duration and network
 * latency.
 */
app.use((_req, _res, next) => {
  setTimeout(next, performanceConfig.serverDuration);
});

/**
 * Compression of response bodies.
 * @see https://www.npmjs.com/package/http-compression
 */
app.use(
  compression({
    threshold: 1, // so we can always see it for testing.
    gzip: performanceConfig.enableGzipCompression,
    brotli: performanceConfig.enableBrotliCompression,
  })
);

/**
 * Host the API Routes
 */
app.use("/api", api);

/**
 * Host static assets in the ./public directory
 * @see https://expressjs.com/en/5x/api.html#express.static
 */
app.use(
  express.static(resolve(__dirname, "..", "public"), {
    extensions: ["html"],
    etag: performanceConfig.enable304CachingHeaders,
    lastModified: performanceConfig.enable304CachingHeaders,
    cacheControl: performanceConfig.enableBrowserCache,
    maxAge: performanceConfig.enableBrowserCache ? 7200000 : 0,
  })
);

module.exports = app;
```

### 📡 How Compression Works

1. Client Request includes:  
   Accept-Encoding: gzip, deflate, br, zstd
2. Server Response includes:  
   Content-Encoding: gzip (or br)
3. The response body is then compressed accordingly.

### 💡 Key Takeaways

- Compression is an easy win — just a server setting in most stacks.
- Helps improve not just TTFB but total page weight.
- Brotli is ideal for large payloads, Gzip is solid for all-around use.
