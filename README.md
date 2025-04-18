# 4 - Testing and Tools

## 4.1 - Testing Performance

When testing performance, `how and where we collect data drastically affects the results`. There are three main approaches:

---

### 🧪 4.1.1 - Lab Data

- Performance `tests run in a controlled environment`.
- Usually executed close to the host server (e.g., local dev server).
- Results are `consistent but not reflective of real user conditions`.
- Example: Running Lighthouse locally.

> ⚠️ Good for debugging and regressions, but not representative of actual user experience.

### 🛠️ Simulating Reality in Lab Tests

When collecting lab data:

- Simulate mobile vs. desktop use.
- Consider network conditions.
- Think about device power (not everyone has a fast setup).

> 🧠 Lab data must mimic real users to be meaningful.

---

### 🤖 4.1.2 - Synthetic Data

- `Tests are run on remote devices/robots simulating user visits`.
- Crosses real networks, so it's more realistic than lab tests.
- `Still uses high-end devices and fast connections`, which skews results.

> 🌐 Useful for monitoring in production-like environments, but not as accurate as real-world usage.

---

### 👥 4.1.3 - Field Data (Real User Monitoring - RUM)

- `Metrics are collected from real users visiting the site`.
- Most accurate reflection of actual user experience.
- Captures a wide range of devices, networks, and conditions.

> ✅ Essential for understanding how real people experience your site.

---

### 🧠 Key Differences

| Method     | Source                   | Accuracy | Sample Size     |
| ---------- | ------------------------ | -------- | --------------- |
| Lab Data   | Controlled test device   | Low      | Single sample   |
| Synthetic  | Remote scripted bots     | Medium   | Limited samples |
| Field Data | Real users in production | High     | Large dataset   |

> 📊 Lab data gives you one controlled result. Field data gives you thousands of real-world scores.

---

## 4.2 - Understanding Metrics and Percentiles

To interpret performance data meaningfully, we must go beyond averages and dive into percentiles.

---

### 📉 Why Averages Can Be Misleading

Averages tend to oversimplify. For example:

- If scores are: 99, 90, 70, 60 → the average = 80
- But:
  - Half of the users had a great experience.
  - Half had a poor one.
  - No one actually had an “80” experience.

This hides real user experiences.

Another case:

- Most users scored ~85–90.
- But 10% had _terrible_ experiences.
- Still, the average might remain 80.

> 🚫 Conclusion: Averages hide outliers and fail to represent majority or worst-case user experiences.

---

### 📊 Percentiles: A Better Approach

Instead of asking “what’s the average?”, ask:

> What do most users experience?  
> What do the _worst_ users experience?

#### 🔢 Definitions:

- p50 (50th percentile): The median score. Half of users scored below, half above.
- p75 (75th percentile): 75% of users had a better or equal score.
- p95 / p99: Represent the worst 5% or 1% of user experiences.
- Not using p100: it often includes garbage outliers (e.g., "3-year load times").

![](https://i.imgur.com/M2CD79B.png)

> ✅ Google’s Core Web Vitals use p75 for performance scoring.

---

### 📌 Example Distribution

**Even distribution:**

- Scores: 0, 10, 20, ..., 100
- p50 = 50, p75 = 75, p95 = 95
- Average = 50

**Real-world skewed distribution:**

- Most users: 79–85 ms
- Few users: 256 ms (garbage outlier)
- p50 and p75 remain consistent
- But the average shifts upward due to the outlier

> 🎯 Percentiles are stable even when averages get distorted by a few extreme cases.

---

## 4.3 - Google Lighthouse & Performance Panel Deep Dive

### 📊 Lighthouse Overview

- Built into Chrome DevTools as a top tab.
- Generates a familiar Performance Score (green/orange/red).
- Evaluates:
  - Performance ✅ _(focus of this course)_
  - SEO
  - Accessibility

> The Web Vitals extension is no longer available. Google now recommends the DevTools tab Performance.

---

### ⚙️ Testing Setup Tips

To simulate a real-user experience:

- Pop out DevTools to avoid shrinking the viewport.
- Use Responsive Mode and select a real or custom device (e.g., _iPhone 12 Pro_).
- Example:
  - Network Throttling (e.g., Slow 4G)
  - CPU Throttling to mimic mid-to-low tier hardware.

![](https://i.imgur.com/7g8Dd5X.png)  
![](https://i.imgur.com/vfm3PQs.png)

---

### 🚦 Running a Lighthouse Audit

- Select Mobile, apply throttling manually for better realism.
- Metrics Lighthouse evaluates:
  - FCP (First Contentful Paint)
  - LCP (Largest Contentful Paint)
  - CLS (Cumulative Layout Shift)
  - ⚠️ INP (Interaction to Next Paint) not included unless actual interaction happens.

---

### 🎬 Extra Insights From Lighthouse

- Filmstrip: Shows visual progression of page load.
- Diagnostics: Highlights render-blocking resources, oversized images, and other bottlenecks.
- Trace view: Links into the Performance tab for deeper analysis.

---

### 🔍 Performance Tab: Waterfall & Flame Chart

Lighthouse gives a 10,000-foot view, but the Performance Panel is `where devs dig into details`.

### Waterfall Chart

- `Visualizes network requests` over time.
- You can zoom into specific ranges to inspect:
  - Blocked requests
  - Long connection/setup times
  - Render-blocking resources (e.g., CSS)
- Example: Image blocked for 3.3s before download began.
- Highlights high-priority files like fonts or CSS that delay first paint.

### 🔥 Flame Chart

- `Shows CPU tasks during load` (parsing, rendering, executing JS).
- `Very low-level`: requires zooming way in (e.g., 100ms window).
- Helps identify long-running tasks or JS that delays paint.
- Navigable with keyboard (WASD keys) like a game.

---

### 🧠 Developer Pro Tip

- If you're seeing performance issues, use Lighthouse to guide you.
- But to diagnose real problems, jump into the Performance panel.
- `Most performance issues are not JS memory leaks` — they're often about:
  - HTML structure
  - Image sizes
  - Critical resource order
  - Network bottlenecks

---

## 4.4 - Chrome User Experience Report (CrUX)

### 🌍 Real-World, Field-Based Data

The Chrome User Experience Report (CrUX) is `Google’s dataset of real user performance data`, collected from people browsing the web using Chrome while signed into their Google account.

#### 📊 Key Characteristics

- Field Data: Unlike synthetic tests like Lighthouse, CrUX `captures real performance data from actual users`.
- Logged-in Chrome Users Only: Data is `collected from users signed into Chrome with a Google account`. This is consented to via Google’s terms of service.
- Top 1M+ Public Websites: CrUX data is only collected for popular, public domains—`no intranet or localhost data is included`.
- Anonymous & Public: Google anonymizes and obfuscates the data before publishing it (e.g., rounding, fuzzing).
- 28-Day Rolling Average: Data is updated daily, but each day's value reflects the last 28 days' performance.
- Available via:
  - Google BigQuery
  - CrUX API
  - PageSpeed Insights
  - Google Search Console
  - 3rd-party tools (e.g., Speed Check by Request Metrics)

#### 💡 Why CrUX Matters

- It’s the actual data Google uses to `determine if your site deserves performance-based ranking boosts or penalties`.
- It reflects real device performance under real network conditions.
- You `can check your competitors’ scores`, not just your own—useful for benchmarking.

### CrUX vs Real User Monitoring (RUM)

| Feature              | CrUX                                     | Real User Monitoring (RUM)                       |
| -------------------- | ---------------------------------------- | ------------------------------------------------ |
| **Data Type**        | Field Data                               | Field Data                                       |
| **User Base**        | Logged-in Chrome users                   | All users on your site                           |
| **Site Coverage**    | Top 1M+ public websites                  | Any site (including private/internal)            |
| **Data Visibility**  | Public and anonymous                     | Private and detailed                             |
| **Granularity**      | Aggregated, anonymized                   | High resolution (page, session, device, etc.)    |
| **Frequency**        | 28-day rolling average                   | Real-time or near real-time                      |
| **Customization**    | None                                     | Fully customizable (metrics, dimensions, alerts) |
| **Storage & Access** | Google BigQuery, PageSpeed Insights, API | Your own dashboard, backend, or monitoring tools |
| **Ideal For**        | SEO benchmarking and industry comparison | In-depth analysis and user-centric diagnostics   |

---

### 🧪 CrUX vs. Lighthouse

| Feature          | Chrome User Experience Report (CrUX) | Google Lighthouse            |
| ---------------- | ------------------------------------ | ---------------------------- |
| Data Type        | Field (Real users)                   | Synthetic (Simulated test)   |
| Scope            | Public websites only                 | Any site (even localhost)    |
| Device & Network | Real devices, real networks          | Simulated environment        |
| User Interaction | Includes INP                         | No real interaction captured |
| Frequency        | 28-day rolling average               | Instant, on-demand           |
| Public?          | Yes                                  | Only local unless shared     |

---

### 🔍 Using CrUX

#### ✅ Tools to Access CrUX

- [PageSpeed Insights](https://pagespeed.web.dev)
  - Shows both CrUX and synthetic Lighthouse data.
  - CrUX is shown at the top — _real user experience data matters more_.
- Google Search Console
  - Provides CrUX-based Core Web Vitals reports for your own verified properties.
- BigQuery Dataset
  - Publicly accessible if you’re familiar with SQL and Google Cloud (⚠️ querying large datasets may incur cost).
- Speed Check by Request Metrics
  - Simple tool built by Todd Gardner (instructor of the course) for quick CrUX checks on any domain.
  - Allows benchmarking against competitors

---

## 4.5 - WebPageTest.org

[WebPageTest.org](https://webpagetest.org) is a powerful synthetic performance testing tool maintained by Catchpoint. Unlike PageSpeed Insights or Lighthouse, which are often used for quick audits, WebPageTest `provides a much more detailed and configurable testing environment` for advanced diagnostics.

### Key Features

- Custom Test Configuration  
  `Choose location, device, browser, connection type (e.g., 3G, 4G, Cable), and more to simulate real-world conditions`.
- Filmstrip View  
  `Visualizes your page load frame by frame` to analyze paint timing and visual completeness.

- Waterfall Charts  
  Detailed request-by-request breakdown of everything that loads on the page, with blocking, DNS, SSL, and other network phases clearly marked.

- Core Web Vitals Support  
  Recently added support for INP in addition to LCP, CLS, and other metrics.

- Scripting and Custom Steps  
  Automate interactions like login flows, clicking buttons, and waiting for elements—ideal for measuring INP or multi-step apps.

- Repeat View & Cache Testing  
  Run tests with an empty or primed cache to observe differences in load behavior.

- Web Vitals Breakdown by Element  
  Pinpoint the exact image, script, or style responsible for a poor LCP or CLS.

### Use Cases

- Testing third-party impact (ads, tag managers)
- Pinpointing critical rendering path bottlenecks
- Verifying optimizations before release
- Benchmarking competitor performance under identical conditions

### Pro Tips

- Use Lighthouse tab inside WebPageTest for even more insight (combines synthetic and Lighthouse audit).
- Enable video recording and layout shift visualization to track janky user experiences.
- Export data as JSON or HAR for offline analysis or integration into performance monitoring systems.

---
