# Capstone Report

**Author:** Hadia Fatima

**Lane:** Ranking Signal Analysis

**Repo:** https://github.com/hadiafatima007/flyrank-ai-ml-internship

**Date:** 19 July 2026



---

## 1. Problem framing

**Decision this supports:**
This project helps identify which webpages should be prioritized for content optimization based on their search performance signals.

**Unit of analysis:**
An individual webpage.

**Output:**
A ranking signal report showing which factors are associated with better search visibility, along with ranked recommendations for content improvement.

**Action a human takes:**
A content editor or SEO analyst can use the recommendations to decide which pages should be updated, monitored, or optimized first.

**Cost of a wrong call:**
If the model incorrectly identifies important pages, time and resources may be spent improving low-impact pages while missing pages with greater optimization opportunities.

**Why data/ML helps:**
Search performance depends on many interacting factors. Machine learning can analyze these patterns more effectively than manual inspection and provide evidence-based recommendations.

---

## 2. Data safety

**Dataset used:**
This project uses the FlyRank ML Internship dataset provided for the Search Intelligence capstone.

**Columns excluded:**
Client names, domains, URLs, private search queries, credentials, and any client-identifying information will be excluded to comply with the public data policy.

**Leakage risks considered:**
Label-derived fields such as `trend_direction` and `trend_pct` will not be used as input features because they could leak information about the target. Pseudonymous IDs will only be used for grouping during validation and never as model features.

**Privacy confirmation:**
No client-identifying information will appear in this repository or the final research paper.
