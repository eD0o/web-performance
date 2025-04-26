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

---

## 6.3 - Efficient Protocols

Modern websites rely on fast, reliable data delivery. Over time, web protocols have evolved to reduce latency and improve performance.

![](https://i.imgur.com/f85DaVm.png)

### HTTP/1.1

- The traditional, "chatty" web protocol.
- Each asset (HTML, CSS, JS, etc.) `requires a separate request and response`.
- High overhead from repeatedly setting up and tearing down connections.

### HTTP/2

- `Uses a single TCP connection to stream multiple requests/responses`.
- Reduces overhead by reusing the connection.
- More efficient than HTTP/1.1, but `still uses TCP, which requires extra steps` like handshakes and acknowledgments (ACKs).

### HTTP/3

- Built on QUIC, a protocol that runs over UDP instead of TCP.
- Handles secure setup (like TLS) more efficiently.
- No need to wait for ACKs — `data is sent continuously, which makes it faster in most cases`.
- `Offers significant performance gains over HTTP/2`, especially on complex or global websites.

### Real-world Testing

- Tests from servers in Los Angeles, Singapore, and Frankfurt showed:
  - ~2x speed improvement moving from HTTP/1 → HTTP/2 → HTTP/3.
  - Faster load times especially for more complex apps (like SPAs).
  - Some outliers exist, but overall, performance improves with each protocol upgrade.

![](https://i.imgur.com/lgfj765.png)

### 6.3.1 - Real-World Challenges & Demos

While HTTP/2 and HTTP/3 offer major performance improvements, they're not always easy to implement — especially in local development or corporate environments. Here are a few key challenges:

- `Local setup is hard: Simulating a proper HTTP/3 environment requires HTTPS, valid TLS certificates, and often complex proxy configurations`. This makes it tricky to demo or develop locally.
- HTTPS requirement: Both HTTP/2 and HTTP/3 require TLS, which means dealing with certificates — something that’s tedious to manage on local machines.
- Firewall and UDP issues: HTTP/3 runs over UDP, which may require special firewall rules, especially in larger organizations.
- Tooling support is limited: Some tools (like curl) don't fully support HTTP/3 yet. You may need custom builds like curl3 to test things properly.

#### Alt-Svc Header Behavior

When visiting the site:

- The initial request typically uses HTTP/2.
- The server responds with an Alt-Svc header saying “I also support HTTP/3”.
- The browser upgrades future requests to HTTP/3 without user intervention.

This upgrade allows for faster, parallel streaming of assets, even if the Time to First Byte (TTFB) isn't drastically reduced. What matters is that resources are streamed more efficiently, especially under high load or for Single Page Apps (SPAs).

---

# 6.4 - Host Capacity & Proximity

> 💬 _"Right-size your host and bring it closer to users. TTFB is heavily impacted by server capacity and distance."_ — Todd Gardner

## 6.4.1 - Right-sizing Your Host

- Objective: `Ensure your hosting platform has sufficient capacity for your application's workload`.
- Context: Even if metrics (CPU, memory, bandwidth) seem low, under-provisioning can delay server response.
- Example:
  - Todd's app runs on a small DigitalOcean box.
  - Artificial server delay was set to 1000ms.
  - A realistic server processing time would be 30–50ms.
- Action: Reduce server-side processing delay to match real-world needs → significantly improve Time To First Byte (TTFB).
- Result:
  - Pre-optimization: >1s TTFB.
  - Post-optimization: ~0.21s TTFB.
  - Improved First Contentful Paint (FCP) and Largest Contentful Paint (LCP) too.

## 6.4.2 - Network Distance Penalty

After compressing assets, fewer bytes are transmitted over the network.
Using an efficient protocol minimizes unnecessary communication, and ensuring the server has sufficient capacity prevents delays. `However, if the server is located far from the user, additional latency becomes unavoidable`.

- Problem: `Hosting far from users adds unavoidable latency`.
- Process:
  - Server → Local regional network → Global Internet backbone → User’s regional network → Final delivery.
- Example:
  - From Minneapolis to Amsterdam: 117ms minimum latency (measured via [WonderNetwork](https://wondernetwork.com/)).
  - This is a hard physical limit that cannot be optimized away without relocating the server closer to the user.

## 6.4.3 - Solution: Use a CDN (Content Delivery Network)

- How it works:
  - First request → Hits CDN → Cache miss → Pulls from origin (full latency).
  - `Subsequent requests → Served from nearest CDN edge (low latency)`.
- Example:
  - Initial page load (cache miss): 308ms.
  - Subsequent load (cache hit): 63ms.
- Benefits:
  - Users avoid long cross-continental hops.
  - Fast, local copies of content dramatically speed up user experience.
- Tools:
  - Todd used BunnyCDN (but any CDN like Cloudflare, Fastly, etc. can achieve similar results).

## 6.4.4 - Final Results After Improvements

- Setup:
  - Server capacity properly sized.
  - Assets compressed (gzip/Brotli).
  - Efficient protocols (HTTP/2 or newer).
  - CDN deployed for geographic proximity.
- Performance Gains:
  - TTFB: ~0.02s (amazing!).
  - FCP and LCP moved into "green" ranges without even optimizing JavaScript or HTML yet.

## 6.4.5 - Key Takeaways

| Step                    | Impact                            |
| :---------------------- | :-------------------------------- |
| Compress responses      | Fewer bytes to transfer           |
| Use efficient protocols | Reduce chattiness (e.g., HTTP/2)  |
| Right-size your host    | Avoid server processing delays    |
| Use a CDN               | Minimize distance-related latency |

---
