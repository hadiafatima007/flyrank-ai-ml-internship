# Capstone Report —

- **Author:** Hadia Fatima
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/hadiafatima007/flyrank-ai-ml-internship
- **Date:** 2026-08-10

## 1. Problem framing

This project supports the decision of **which content pages should be reviewed first for a possible refresh or optimisation**.

The unit of analysis is a **content page**. The output is a ranked action score and reason code that places pages into a priority queue. A human editor can use the queue to decide which pages to review for content refresh, existing-content optimisation, metadata/CTR improvement, or monitoring.

The cost of a wrong call is mainly wasted editorial effort: a page may be prioritised even though it does not actually need a refresh, while a genuinely useful opportunity may be missed. Data and ML help because the warehouse contains multiple signals about search demand, ranking, traffic, engagement, content age, and freshness. A model can combine these signals consistently and produce a ranked queue that can be compared with a transparent rule.

This is **decision-support**, not an automated claim that a page will definitely decline or that changing a page will cause a particular search outcome.

## 2. Data safety

The analysis uses the anonymized FlyRank content-refresh dataset used in the weekly assignments. The working starter dataset contains **30,000 rows and 44 columns**, covering **32 pseudonymous clients**.

The analysis uses content/search, traffic, engagement, freshness, and content metadata fields. The pseudonymous `client_id` is used only for the grouped train/test split and is **not used as a model feature**.

The following fields were deliberately excluded from the Logistic Regression features:

- `trend_direction` — this defines the observed decline target.
- `trend_pct` — this is directly related to the observed trend outcome.
- `impressions_last_30d` — excluded because it contributes directly to the observed 30-day trend comparison.
- `impressions_prev_30d` — excluded for the same leakage concern.
- The baseline's own score, reason code, and action outputs — these are decision outputs, not predictive inputs.

The target was defined as:

> `1` when `trend_direction == "down"`, otherwise `0`.

The model therefore predicts an **observed decline label**, not a future causal effect.

The release exposes 90-day aggregate performance fields and a last-30-days versus previous-30-days comparison. The completed notebooks do not establish calendar start/end dates for the release, so no calendar date window is invented here.

The data is anonymized. No client names, domains, URLs, private queries, credentials, or raw client exports are included in the analysis outputs. Pseudonymous IDs are used only where necessary for grouping and identification inside the dataset.

## 3. Baseline

The first approach was a transparent rule-based action score.

The baseline gives points for four signals:

- **Staleness:** `days_since_last_update > 180` → +40
- **Search demand:** `search_volume >= 100` → +30
- **Ranking opportunity:** `avg_position` between 8 and 20 → +20
- **CTR opportunity:** `ctr < 2` → +10

The resulting score creates a ranked review queue.

The baseline reason codes map signals to actions:

| Reason code | Meaning | Action |
|---|---|---|
| `STALE_REFRESH` | Old content with a refresh opportunity | Refresh Content |
| `QUICK_WIN` | Ranking/search-demand opportunity | Optimise Existing Content |
| `CTR_FIX` | Low CTR opportunity | Improve Title & Meta |
| `LOW_PRIORITY` / `MONITOR` | Does not meet the stronger priority conditions | Monitor |

The baseline is a fair comparison because it is transparent, deterministic, and was evaluated on the **same held-out test data and the same Precision@50 metric** as the Logistic Regression model.

On the client-grouped test split, the baseline achieved:

**Precision@50 = 0.54**

The test-set decline base rate was:

**51.1%**

Therefore the baseline's top-50 precision was above the overall decline rate, but it was still treated as a simple benchmark rather than as a validated causal rule.

## 4. Model / analysis

The selected method is **Logistic Regression**.

It fits the lane because the task is to identify pages associated with the observed decline label and then use the predicted probability to **rank pages for review**. Logistic Regression is also interpretable, making it suitable as a first model before trying more complex methods.

The numerical features used were:

- `search_volume`
- `competition`
- `cpc`
- `word_count`
- `char_count`
- `impressions_90d`
- `clicks_90d`
- `pageviews_90d`
- `sessions_90d`
- `users_90d`
- `engaged_sessions_90d`
- `ai_sessions_90d`
- `scroll_events_90d`
- `days_with_impressions`
- `days_with_sessions`
- `content_age_days`
- `age_tier_order`
- `days_since_last_update`
- `ctr`
- `avg_position`
- `engagement_rate`
- `scroll_rate`
- `ai_traffic_pct`

The categorical features used were:

- `competition_level`
- `content_type`
- `main_intent`
- `provider_used`

Numeric missing values were median-imputed and standardized. Categorical missing values were filled with the most frequent value and then one-hot encoded. The classifier was Logistic Regression with `max_iter=1000`.

The target/proxy definition is:

> The target is 1 for pages whose observed `trend_direction` is `"down"` and 0 otherwise.

The model deliberately does **not** use the observed trend fields or the 30-day impression fields that could reveal the outcome.

## 5. Evaluation

The evaluation uses a **grouped train/test split by `client_id`**.

There are 32 clients in total. The split produced:

- **25 training clients**
- **7 test clients**
- **23,837 training rows**
- **6,163 test rows**
- **0 client overlap**

The split uses `random_state = 42` so it can be reproduced.

Grouping by client is intended to test whether the model can generalise to clients that were not represented in training, rather than benefiting from near-duplicate client-specific patterns across both sets.

Both the baseline and model were evaluated on exactly the same held-out test clients using Precision@50.

| Method | Precision@50 | Test decline base rate |
|---|---:|---:|
| Week-4 baseline | 0.540 | 0.511 |
| Logistic Regression | 0.720 | 0.511 |

The Logistic Regression model therefore measured **18 percentage points higher Precision@50** than the baseline on this held-out client split.

The model's top-50 selection contained **14 false positives**. These are pages that the model ranked in its top 50 but whose observed test target was not decline. This shows that the ranking is useful but imperfect.

The result should not be interpreted as proof of future performance. It is an observed result on one fixed client-grouped test split.

## 6. Interpretation

The largest absolute Logistic Regression coefficients in the completed analysis included:

| Feature | Coefficient |
|---|---:|
| `users_90d` | -0.988 |
| `sessions_90d` | +0.770 |
| `days_with_impressions` | +0.604 |
| `main_intent_navigational` | -0.450 |
| `days_with_sessions` | -0.427 |
| `content_type_keyword article` | +0.357 |
| `content_type_feedly article` | -0.340 |
| `content_age_days` | -0.288 |
| `scroll_events_90d` | +0.281 |
| `word_count` | +0.243 |

The model therefore leaned strongly on historical traffic/activity signals, including users, sessions, and the number of days with impressions or sessions. Content characteristics also contributed to the ranking.

The important interpretation is **directional rather than causal**. A positive or negative coefficient describes how the feature is associated with the model's predicted probability after preprocessing; it does not show that changing that feature will cause content to decline or recover.

A useful negative result is that the baseline signals should not be treated as guaranteed causes of decline. The Week-4 signal checks supported using freshness and search demand as prioritisation signals, but the later model shows that the final ranking also depends on several historical activity and content features.

The false positives are also important. For example, some highly ranked pages had high model scores even though the observed target was 0. This means the model should be used to create a **review queue**, not an automatic action system.

## 7. Recommendation

The output supports a practical editorial queue:

| Priority/action | Use |
|---|---|
| **Refresh Content** | Review stale pages with stronger evidence of decline/opportunity. |
| **Optimise Existing Content** | Review pages with useful search demand and ranking opportunity. |
| **Improve Title & Meta** | Review pages where low CTR makes metadata optimisation worth checking. |
| **Monitor** | Keep weaker-signal pages under observation rather than spending immediate editorial effort. |

A FlyRank editor could start with the highest-ranked pages, inspect the reason code and underlying signals, and then decide whether the page genuinely deserves intervention.

The model's **0.72 Precision@50** makes it a stronger measured ranking tool than the **0.54 baseline** on this test split. However, the confidence should remain limited to **decision-support** because:

- the evaluation uses one fixed client-grouped split;
- the target is an observed trend label rather than a future experimental outcome;
- the model has false positives;
- the data is anonymized;
- the analysis does not establish causal effects of refreshing content;
- the model does not prove anything about Google's ranking algorithm.

The recommended workflow is therefore:

**Model ranks → human reviews → editor decides action → outcome is monitored.**

The ranked recommendations should be treated as opportunities for investigation, not guaranteed fixes.

## 8. Reproducibility

The analysis should be rerun from a fresh clone using the repository's documented environment.

Core environment:

- Python
- pandas
- numpy
- scikit-learn
- pathlib
- DuckDB where used by the earlier data workflow

The model uses:

```text
random_state = 42
```

The evaluation uses:

```text
GroupShuffleSplit(
    n_splits=1,
    test_size=0.20,
    random_state=42
)
```

The repository should contain:

```text
work/
├── notebooks/
│   ├── ML-04...
│   ├── ML-05...
│   ├── ML-06...
│   └── ML-07...
├── outputs/
│   └── baseline_action_score.csv
└── capstone_report.md

submission/
└── paper_url.txt
```

The exact notebook filenames should match the files committed in the repository.

The baseline output is written to:

```text
work/outputs/baseline_action_score.csv
```

The final paper URL must be stored as the only line in:

```text
submission/paper_url.txt
```

A pinned `requirements.txt` should also be committed so that the environment used for the final run can be reproduced.

Before submission, run the notebooks from top to bottom and verify that the reported metrics still match the fresh run.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support **Metrics vs. base rate:** the test decline base rate is 51.1%, reported next to Precision@50. No causal claims are made. No claim is made about predicting Google's algorithm. No client-identifying details, private queries, credentials, or raw exports are included.
