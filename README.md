# 1 - Introduction to Web Performance

It refers to the `speed and efficiency with which a website loads, renders, and responds` to user interactions.

## 1.1 - Why it matters

Instead of only adding more content, improving web performance is essential for three main reasons:

1. User Experience (UX)
2. Search Engine Optimization (SEO)
3. Online Advertising Efficiency

Each of these is critical depending on your website's focus—but `at least one applies to every site`.

### 1.1.1 - User Experience

- Fast sites prevent user frustration — `poor performance makes users angry and likely to leave`.
- Users come with expectations based on past experiences and competitor sites.
- `We can’t control user devices or connections, so we must optimize` our websites instead.

#### Human Perception Benchmarks

| Response Time      | User Reaction                          |
| ------------------ | -------------------------------------- |
| **< 0.01 seconds** | Perceived as instant                   |
| **~1 second**      | Noticeable but does not interrupt flow |
| **> 2 seconds**    | Breaks concentration, feels slow       |
| **> 10 seconds**   | Causes frustration and abandonment     |

#### Real-World Impact

- 40% of users abandon a site that takes longer than 3 seconds to load.
- 75% of users who perceive a site as "slow" will not return.

> Time is money: `faster websites retain users, build trust, and improve conversions`. Ignoring performance means lost engagement and revenue.

### 1.1.2 - SEO (Search Engine Optimization)

- For public-facing sites, `search ranking directly impacts traffic — most users click the first few results`.
- Since 2020, `Google includes Core Web Vitals in ranking signals`, so performance now affects visibility.
- If two sites have similar content and backlinks, the `faster one will usually rank higher`.
- Traffic sharply drops beyond position #3 — `being just a little slower can drastically reduce visitors`.

![](https://api.backlinko.com/app/uploads/2022/08/the-number-one-result-in-google-has-the-highest-organic-ctr-1280x988.webp)

#### Click-Through Rate by Position

| Position | Average CTR (example)                      |
| -------- | ------------------------------------------ |
| **#1**   | ~75 million clicks (for high-volume terms) |
| **#2**   | ~10% of #1                                 |
| **#3**   | ~50% of #2                                 |
| **>10**  | Negligible                                 |

> Core Web Vitals like LCP and CLS are critical. Sites in top positions usually have better performance scores.

### 1.1.3 - Online Advertising & Bounce Rate

- When you pay for ads, `you pay for impressions and clicks, not guaranteed engagement`.
- `Poor performance causes high bounce rates` — users leave before the page loads or becomes usable.
- In an example: `$1,000 in ads → 1,600 clicks → 60% bounce → only 640 real shoppers`.
- By improving performance and lowering bounce by 20%, you'd get 832 shoppers for the same spend — that’s 192 more shoppers and a 25% lower cost per shopper.

#### Bounce Rate vs Performance

| Performance Impact | Result                                  |
| ------------------ | --------------------------------------- |
| +65% faster site   | -20% bounce rate, +200% time on page    |
| Walmart Example    | +100ms = **+1% revenue**                |
| Skill.co Example   | +1 sec = **-0.4% conversion rate drop** |

> Performance isn't just technical — it has real revenue impact.

## 1.2 - Measuring

Without measurement, performance efforts are blind guesses, and `it goes beyond just load time and reflect real user experience`.

### 1.2.1 - From Legacy to Modern Metrics

| Metric Type         | Description                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------- |
| **Legacy**          | Traditional metrics like `"Load Time" or "DOMContentLoaded" — simple but limited.`        |
| **Core Web Vitals** | Google's standardized UX-focused metrics: `LCP, FID, CLS`. Crucial for SEO & performance. |
| **Other Metrics**   | Include `TTFB (Time to First Byte), TTI (Time to Interactive)`, etc. Still useful today.  |

### 1.2.2 - Waterfall Charts

Waterfall charts visualize how each resource is loaded in the browser, it's `useful for identifying bottlenecks, delays, or blocking assets`.

### 1.2.3 - Anatomy of a Waterfall Row (Example)

A simple chart:
![](https://i.imgur.com/O0SMbWg.png)

Now, a more detailed one:

![](https://i.imgur.com/CoxAaOu.png)

Horizontal axis = Time; Vertical = Requests.

| Resource Type | Color  |
| ------------- | ------ |
| HTML          | Blue   |
| CSS           | Purple |
| JavaScript    | Yellow |
| Images        | Green  |
| Fonts         | Teal   |
| Other         | Gray   |