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

