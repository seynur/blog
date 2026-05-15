---
layout: default
title:  "📘 Part 1: Behavioral ML for Windows Authentication: Concepts & Engine"
summary: "ML Sidecar is an experimental Python engine that models “normal” authentication behavior, scores deviations from multiple perspectives, and turns them into flexible, explainable outlier signals for Splunk."
author: Öykü Can
image: /assets/img/blog/2025-12-24-building-a-behavioral-ml-engine-for-windows-authentication-logs.webp
date: 24-12-2025
tags: splunk python windows ml anomaly
categories: Splunk 

---

# 📘 Part 1: Building a Behavioral ML Engine for Windows Authentication Logs  
## 🌸 1. Architecture, Features, Clustering, Scoring, Thresholds & Drift Detection

Modern authentication telemetry can be quite a handful, generating lots of data with a lot of variability! A single user might log in from different devices, networks, and endpoints, and service accounts often show repetitive behaviors that can still be noisy. This presents a challenge because attackers can take advantage of this complexity, trying to mimic normal activity while slipping just past traditional security rules.

I started to think: what if I could model what “normal” authentication behavior looks like using clusters? Then, I could score any deviations in a way that’s easy to explain and works smoothly with Splunk. This little idea blossomed into ML Sidecar—a handy Python engine that runs an unsupervised pipeline for Windows authentication events. It generates understandable anomaly signals and sends the results back to Splunk without adding any new data (just using KVStore).

> ⚠️ **A quick note about the data**: I designed and tested this project using synthetic authentication logs. Synthetic data is usually cleaner than what you'll find in real enterprise environments since it doesn't deal with issues like misconfigurations, inconsistent field quality, timezone problems, or missing assets. The model design is solid, but it’s a good idea to double-check the thresholds and weights against actual production data.

### 1.1. **Repository & Code Map**

Feel free to join us in exploring the repository!

📁 **Repository:**  
[Check out the GitHub - seynur-demos](https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_ml_sidecar)

Here are some key engine modules you'll want to check out:
- ml_sidecar/core/features.py
- ml_sidecar/core/model.py
- ml_sidecar/core/pipeline.py
- ml_sidecar/core/profiles.py
- ml_sidecar/core/kvstore.py
- ml_sidecar/run_auto.py

In *Part 1*, we'll be diving into the interesting internal workings and concepts of the engine.  
Then, in *Part 2*, we’ll move on to execution, Splunk integration, KVStore writes, and the creation of dashboards.

Now that we’ve covered the motivation and the scope of our journey, let’s jump into the architecture!

---

## 🧩 2. Architecture Overview

The ML Sidecar is designed to be a friendly "*companion engine*" for your data needs!
- Think of Splunk as your go-to source for raw events. 
- Meanwhile, Python takes care of the heavy lifting with modeling and scoring, all outside of the Splunk search runtime.
- When it's time to share results, they’re easily exported to KVStore collections, making it super quick to access them for dashboards and investigations.

```
        Raw Authentication Logs
                 ↓
     Normalization / JSON Conversion
                 ↓
      Splunk SPL (light filtering only)
                 ↓
──────────────────────────────────────────
      ML Sidecar (Python ML Engine)
──────────────────────────────────────────
   1. Feature Extraction
   2. User Profile Modeling
   3. Feature Scaling
   4. Auto-K KMeans Clustering
   5. Structural Outlier Scoring (Centroid Distance)  
   6. Contextual & Rarity Scoring
   7. Final Hybrid Anomaly Thresholding
   8. Drift Detection (Chi-Square)   
   9. KVStore Export
──────────────────────────────────────────
                 ↓
        Splunk KVStore Collections
            - auth_events
            - auth_user_profiles
            - auth_cluster_profiles
            - auth_user_thresholds
            - (optional) auth_outlier_events
                 ↓
        Dashboards & SOC Analysis
```

This clear division of tasks helps keep the Splunk app straightforward while giving the ML engine all the flexibility it needs!

---

## 🧮 3. Feature Engineering (Extracted → Computed → Learned)

We transform authentication logs into a neat numerical feature vector! The table below showcases all the features we use for model training and inference, making it easy to understand. It directly connects with the pipeline diagram from *Section 2*, so you can see how data flows and what goes into the machine learning model. Plus, if you’re interested, you can find a detailed diagram in the [ml_sidecar/README.md](https://github.com/seynur/seynur-demos/blob/main/splunk/splunk_ml_sidecar/ml_sidecar/README.md) file. Enjoy exploring!

| ![screenshot](/assets/img/blog/2025-12-24-model-feature-table-set.webp) |
|:--:| 
| *Figure 1* Table of the Sidecar model feature set. |

---

## 🎯 4. Clustering with Auto-K KMeans
Instead of having to manually pick a number of clusters, our system does the hard work for you by testing out different values for K:

```
K ∈ {6, 8, 10, 12, 14}
```

Here’s how it works for each K:
1. Train the KMeans model.
2. Calculate the silhouette score to see how well the clusters are formed.
3. Toss out any models that collapse into a single cluster where all points belong together.
4. Choose the K that gives the best silhouette score.

***Why Choose KMeans?***

- Authentication behaviors naturally group together (think workstation logins, server logins, VPN connections, privileged logins, and service accounts).
- It’s super quick, allowing us to train the model every day or week without hassle.
- The distance to the centroid gives us a clear signal to spot anomalies.
- The clusters we find are easy to understand, which is great for our Security Operations Center (SOC) teams!

> ***Fallback***: If things don’t work out with the other options, we’ll default to K=4 to make sure we still have a working model. When that happens, it's a good idea to double-check your inputs using some multivariate analysis. 

> ***Note***: If you don't like the KMeans, you can easily switch up the model used in the algorithm by tweaking the `core/model.py` script. 🐣

---

## 🚨 5. Composite Anomaly Score
Capturing the quirky side of "security weirdness" can be tricky with just one metric. That’s why Sidecar uses **four** different signals to give you a clearer picture:

**1. Raw Outlier Score (Structural Deviation)**

```
outlier_score ∈ [0, 1]
```

This score helps answer:
- How different is *this event* from what the model sees as typical behavior?

**2. Cluster Rarity (User-Specific Deviation)**

```
cluster_rarity = 1 − freq(user, cluster) / total(user events)
```

This tells us:
- How unusual is this cluster *for that specific user*?

**3. Signature Rarity (Cluster-Level Deviation)**

```
signature_rarity = 1 − P(signature | cluster)
```

This gives insight into:
- How *uncommon* is the *signature* among its *cluster*?

**4. User Hour Score (Temporal Deviation)** - This measures how the current time differs from what’s typical for the user:

```
user_hour_score = min(|hour − mean_hour| / std_hour, 1)
```

**BONUS: Weighted Final Score** - This produces a score between [0, 1]. Keep in mind, though, that this isn't a final decision—just an indicator! The actual verdict comes from **thresholding**.

```
final_anomaly_score = 0.4 * outlier + 0.3 * cluster_rarity + 0.2 * signature_rarity + 0.1 * user_hour_score
```

> ***Sparkling Note***: All constants in the final_anomaly_score calculations can be changed depending on your data and your own work. I determined these values ​​somewhat empirically while examining the data and conducting this study.

Hope this helps you understand the process a bit better! 🪄

---

## 🐣 6. Per-User Normalization
Users have unique score distributions, so a "*noisy*" service account might stand out as unusual compared to the steady patterns typically seen with human workstations.

To make sense of this, we use a friendly *per-user* normalization process:

1. We start by gathering each user's final anomaly score distribution.
2. Then, we calculate the average and standard deviation.
3. Next, we determine the z-score.
4. Finally, we convert that z-score into a value between 0 and 1 using the sigmoid function, creating what we call the per-user normalized score.

This way, we can easily understand what’s "rare for this user" and compare it across everyone.

---

## 🪄 7. Adaptive Thresholding (User / Cluster / Global)
Instead of sticking to a fixed threshold (like 0.7), *Sidecar* uses percentile-based thresholds to keep things smart and aware of the data distribution!

### 7.1. User Threshold

We calculate a unique threshold for each user based on their normalized scores:
- `per_user_threshold_norm`
- `behavior_outlier_user`

### 7.2. Cluster Threshold

We know that different clusters can vary quite a bit. For instance, "*VPN clusters*" can be a bit noisier than "*service-account clusters*." That’s why we create specific thresholds for each cluster:
- `cluster_threshold`
- `behavior_outlier_cluster`

> (Oh, and if you want, you can set a minimum cluster size to ensure that we only trust the cluster thresholds that have enough data!)

### 7.3. Global Threshold

This is our go-to threshold that applies across the entire system for the final anomaly score:
- `threshold_global` (you'll find this one exported via `auth_user_thresholds`)
- `behavior_outlier_global`

---

## 🔮 8. Final Outlier Decision Logic (Configurable)
Sidecar offers a variety of unique signals to help answer the question, "*Is this an outlier?*" The reason for this diversity is straightforward: anomaly detection often depends on perspective.

You can choose from a few options for defining what an outlier means to you:

1. ***`User Threshold` (Personal Baseline)***: This metric is tailored to each user based on their own score distribution, adjusting for individual patterns of behavior. This means it works really well for users who may show lots of variation in their activity compared to those who are more consistent. 

      ***This helps you determine***: Is this *unusual* for this *specific user*?

2. ***`Cluster Threshold` (Behavior-Type Baseline)***: Clusters represent different patterns of behavior (like VPN use, regular workstation activity, or service accounts). Since some clusters naturally exhibit more variability, this threshold allows for some noise while still highlighting rare events in more stable clusters.

      ***This helps you determine***: Is this *unusual* for this *type of behavior*?

3. ***`Global Threshold` (Environment-Wide Baseline)***: This option provides a straightforward cutline across the entire system. It’s particularly handy when you’re dealing with new users or when the clusters are frequently changing.

      ***This helps you determine***: Is this *rare* in *the whole environment*?

For more flexibility, you can also opt for a combined approach:
1. **`Combined Threshold`**: You can choose which thresholds to use and how to combine them.

      If you set the outlier mode to "combined," you can enable or disable each threshold source based on your needs. 
      
      For example:
      - User + Cluster (ignore global): Great for when you want the “personal” and “behavior-type” baselines to align.
      - User + Global (ignore cluster): Ideal if you trust individual user profiles but still want a system-wide check.
      - Cluster + Global (ignore user): Useful when user history is limited, but you have strong behavior clusters.

      Then, you can choose how these combined thresholds work:
      - **Combined Logic: "and"** → This is stricter with fewer alerts (you get higher confidence).
      - **Combined Logic: "or"** → This is more sensitive and yields more alerts (you get higher recall).

*Example configuration*:

```
modeling:
  outlier_mode: "combined"     # user | cluster | global | combined

  # used when outlier_mode = combined
  combined_logic: "and"        # and | or
  thresholds:
    enable_user: true
    enable_cluster: true
    enable_global: false
```

Each threshold captures different types of rarity since they are based on distinct distributions:
- A user can be “weird” compared to their own typical behavior, even if similar patterns are common globally.
- Clusters might be noisy, so the global rarity could flag too many issues—this is where cluster thresholds come in handy.
- A global threshold can help identify rare spikes even when a user’s data is still developing.

> *Keep in mind*: User thresholds are genuinely unique for each person—meaning each user can have their own specific cutoff. This customization is essential, especially when users have very different behaviors, like admins, service accounts, regular employees, or shared accounts.

---

## 🔁 9. Drift Detection (Behavior Change Monitoring)
Authentication behavior can vary with the seasons or due to operational changes like VPN usage, migrations, or password updates. To keep up with these changes, our engine uses a Chi-Square (χ²) comparison to detect any differences: 

```
Compare: old_cluster_dist vs new_cluster_dist

If p-value < drift_threshold: retrain model
```

We typically set our drift threshold at `drift_threshold = 0.05`. 

This method helps keep our model aligned with changing behaviors without requiring any manual adjustments. It's all about making things easier and more efficient!

---
## 💾 10. KVStore Output Model

**Hey there!** 

Sidecar publishes outputs to KVStore collections:

- **`auth_events`**: These are enriched event records that make our dashboards shine!
- **`auth_user_profiles`**: We provide neat summaries of individual user behaviors.
- **`auth_cluster_profiles`**: These offer insights per cluster, including signature distributions.
- **`auth_user_thresholds`**: Daily thresholds are set for users, clusters, and globally.
- **`auth_outlier_events`** (optional): This is where we capture only the outlier events.

**A Quick Note on Design**: To keep everything running smoothly, the engine writes exclusively to KVStore-no index writing here! This smart choice helps make Splunk searches quick and keeps our dashboards zippy. 

> Also, keep in mind that the `auth_events` and `auth_outlier_events` lookups have a lot of data. So, before using this app in a production environment, be sure to adjust the output settings. 

---

## 📘 11. Summary (End of Part 1)

Exploring authentication behavior doesn’t require a massive ML platform or opaque black-box detections. This side project started as a curiosity-driven experiment and gradually turned into a practical way to model how authentication normally behaves—and how it sometimes doesn’t.

In ***Part 1***, we focused on the engine itself and covered:
- 🧩 Turning raw Windows authentication logs into meaningful features
- 🎯 Grouping behavior using adaptive KMeans clustering
- 🚨 Scoring anomalies across structural, behavioral, contextual, and temporal dimensions
- 📊 Using percentile-based user, cluster, and global thresholds instead of hard-coded values
- 🔁 Detecting behavioral drift and retraining automatically when patterns change
- 🧠 Keeping everything explainable, configurable, and dashboard-friendly

Rather than chasing “perfect detections,” the Sidecar is designed to surface interesting deviations—and let analysts decide what matters.

👉 Coming up in ***Part 2***, we’ll shift from concepts to execution and walk through:
- 🧪 Generating realistic synthetic Windows authentication logs
- 📥 Ingesting them into Splunk
- ▶️ Running the ML Sidecar end-to-end
- 🗄 Writing enriched results to KVStore
- 📈 Exploring behavior and outliers through dashboards

> 📁 Curious about the implementation? You can explore the full codebase here: [GitHub Repository](https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_ml_sidecar)  

💬 Have ideas, questions, or thoughts on behavioral modeling in Splunk?
I’d love to hear how you approach authentication anomalies.

Connect with me on [*LinkedIn*](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/) or drop a comment on the blog. 

Until Part 2 👋🔍

---

# References:
- [[1]](https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_ml_sidecar) Seynur (2025). seynur-demos/splunk_ml_sidecar [GitHub repository]. GitHub. https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_ml_sidecar

- [[2]](https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch) Splunk Inc. (2024). Splunk REST API Reference – Search Job Export Endpoint.
https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch

- [[3]](https://docs.splunk.com/Documentation/Splunk/latest/Admin/AboutKVstore) Splunk Inc. (2024). Working with KV Store Collections in Splunk.
https://docs.splunk.com/Documentation/Splunk/latest/Admin/AboutKVstore

- [[4]](https://jmlr.org/papers/v12/pedregosa11a.html) Pedregosa, F., Varoquaux, G., Gramfort, A., et al. (2011). Scikit-learn: Machine Learning in Python. Journal of Machine Learning Research, 12, 2825–2830.
https://jmlr.org/papers/v12/pedregosa11a.html

- [5] Tan, P.-N., Steinbach, M., & Kumar, V. (2019). Introduction to Data Mining (2nd ed.). Pearson.
(Reference for clustering, silhouette score, and distance-based anomaly concepts.)

- [[6]](https://dl.acm.org/doi/epdf/10.1145/1541880.1541882) Chandola, V., Banerjee, A., & Kumar, V. (2009). Anomaly Detection: A Survey. ACM Computing Surveys, 41(3).
https://dl.acm.org/doi/epdf/10.1145/1541880.1541882

---