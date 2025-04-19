# 5 - Setting Performnce Goals

## 5.1 - How fast should your site be?

Determining how fast your website needs to be isn't just about technical speed, `it's about user expectations, perceived value, and context`. Web performance is both a `technical and psychological challenge`.

### 🎯 Key Questions

- How fast is _fast enough_?
- Who defines that speed?
- What do your users expect?

---

### 💡 Performance is Relative

- Speed is subjective and varies based on:
  - The `type of application` (fun vs. functional).
  - The `audience` (impatient developers vs. general users).
  - The `value of the task` (sticker shopping vs. loan application).

For example:

- Developer tools → must feel instant.
- Tax software → `users tolerate slowness if it feels secure and trustworthy`.

---

### 🧠 Human Perception > Raw Metrics

Users often:

- Perceive things as slower than they actually are.
- Remember the slowness worse over time.

> Feeling fast is often more important than what is fast.

---

### 🧪 Psychological Learnings (from 1960s studies)

| Scenario                      | Perception Outcome                      |
| ----------------------------- | --------------------------------------- |
| Bored wait                    | Feels longer                            |
| Anxious wait                  | Feels slower                            |
| Unexplained wait              | Feels longer and frustrating            |
| Uncertain duration            | Feels endless                           |
| Distraction (e.g., animation) | Can shorten perceived wait              |
| Value-based wait              | People accept more delay for more value |

#### Example: TurboTax

TurboTax intentionally slows down the "Submit" step with animations to give users a sense of confidence. `Studies showed users trusted the tool more when they _felt_ like complex things were happening—even when they weren’t`.

---

### ✅ Design Considerations

- `Communicate why the user is waiting`.
- Offer progress indicators or distractions.
- Tailor performance goals based on the perceived value of the interaction.
- Remember: perception = experience.

---

## 5.2 - Determining Performance Goals

`You don’t get to decide what’s “fast enough”—your users, competitors, and SEO do`. Performance goals must be set based on real-world impact and context.

### 1. User Experience

- Focus on business metrics, not just performance scores:
  - Bounce rate, session time, conversion rate, cart abandonment, scroll depth, etc.
- Find correlations between changes in Core Web Vitals (like LCP or CLS) and shifts in business metrics.

> Remember: `correlation ≠ causation` (e.g., pirate population vs. global warming). Always test assumptions.

### 2. Competitors

- Benchmark against similar sites. You need to `be at least 20% faster for users to notice a difference`—this is based on Weber’s Law.

![](https://i.imgur.com/TqywOvp.png)

- Minor improvements (like 4% faster) won’t be perceived; large gaps (like 57%) will.

### 3. SEO Impact

- Google uses Core Web Vitals (LCP, CLS, INP) in rankings.
- Even if your content is excellent, poor performance can hurt discoverability.
- Hitting the Web Vitals thresholds is critical to stay competitive in search results.

Bottom line: Don’t guess—use data, user feedback, and competitive benchmarks to determine how fast your site _needs_ to be.

---

## 5.3 - Understanding your Users

Here’s the summarized section as requested:

---

## 5.3 - Understanding Your Users

To build a performant site, you must first `understand who your users are, what devices they use, and under what conditions they browse`.

### Key Takeaways:

- Mobile dominates: As of August 2024, 62% of global web traffic is mobile, making `mobile performance a critical priority`.
- Device diversity: Screen sizes vary drastically, from tiny Android screens (~360px wide) to small laptops (1366px) and beyond. `Developer screens often don't reflect real user environments`.
- OS usage:
  - On mobile: 71% Android, 27% iOS.
  - On desktop: 71% Windows, with macOS and Linux making up smaller shares.
- Hardware limitations: `The average Android phone globally costs ~$286, meaning most users operate low-powered devices with limited RAM and CPU`.
- Network speed (worldwide averages):
  - Mobile: 60 Mbps down / 11 Mbps up / 27ms latency.
  - These vary greatly by region—some areas exceed 250 Mbps, others fall under 10 Mbps.
- Developer bias: Developers often test on fast, modern devices and networks that don’t match the real-world constraints of actual users.
- Use your analytics: `Tools like real-user monitoring (RUM) and analytics platforms can reveal the actual devices, OSs, browsers, and network conditions your audience uses`. This data should guide your performance priorities and testing environments.

---
