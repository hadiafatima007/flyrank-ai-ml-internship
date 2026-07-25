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

---

## 3. Baseline

**Baseline approach:**
The baseline model will predict the average value of the target variable for every webpage.

**Why it is a fair comparison:**
This simple approach provides a transparent reference point. Any machine learning model should perform better than this baseline to justify its use.

**Evaluation:**
The baseline and the machine learning model will be evaluated using the same train/test split and the same evaluation metrics. Final performance values will be added after model training.

---

## 4. Model / analysis

**Method:**
This project will use supervised machine learning to analyze the relationship between search performance signals and search visibility. The primary model will be selected after exploratory data analysis and compared with a simple baseline.

**Why it fits this lane:**
Ranking Signal Analysis aims to identify which search and content signals are associated with better search performance. A supervised learning approach is suitable for discovering these relationships and supporting data-driven recommendations.

**Planned features:**
The initial feature set may include impressions, clicks, CTR, engagement metrics, and other public-safe search performance features available in the dataset. The final feature list will be determined after exploratory data analysis.

**Features intentionally excluded:**
Client-identifying information, label-derived fields (such as `trend_direction` and `trend_pct`), and pseudonymous IDs will not be used as model features.

**Target definition:**
The final target variable will be selected after reviewing the available dataset and will represent a measurable search performance outcome suitable for the chosen machine learning task.

---

## 5. Evaluation

**Validation strategy:**
The dataset will be divided into separate training and testing sets using an appropriate validation strategy. A time-aware or grouped split will be used if required by the data to reduce leakage and improve the reliability of the evaluation.

**Evaluation metrics:**
The machine learning model and the baseline will be evaluated using the same validation split and the same evaluation metrics. The final metrics will be selected based on the prediction task and may include MAE, RMSE, and R² for regression.

**Error analysis:**
After model training, prediction errors will be analyzed to identify where the model performs well and where it struggles. This analysis will help understand the model's limitations and possible improvements.

---

## 6. Interpretation

After training the model, the results will be interpreted to identify which search signals have the strongest relationship with search visibility.

Feature importance or other model interpretation techniques will be used to explain the model's decisions in simple terms.

Any unexpected findings, weak relationships, or negative results will also be documented, as these are valuable for understanding the dataset and the model's limitations.

---

## 7. Recommendation

The final output will provide ranked recommendations to help prioritize content optimization efforts.

Based on the model's predictions and analysis, pages may be recommended for updating, monitoring, improving, or maintaining.

These recommendations are intended to support editorial decisions rather than replace human judgment.

The confidence and limitations of the recommendations will be clearly stated in the final report.

---

## 8. Reproducibility

The complete workflow will be documented so that the project can be reproduced from a fresh clone of the repository.

The final report will include:
- Commands to run the notebooks.
- Random seeds used during model training.
- Python package requirements.
- Any additional setup instructions required to reproduce the results.

---

## Claims checklist

- Results will be reported as observed and measured.
- No causal claims will be made without experimental evidence.
- No client-identifying information will be included.
- The machine learning model will be compared fairly against the baseline using the same validation split.
- All reported results will be reproducible from the repository.
