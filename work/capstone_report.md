# Content Decay & Refresh Prioritization

**FlyRank ML Internship Capstone — Refresh / Content Opportunity Scoring**  
**Author:** Hadia Fatima

## Abstract

Which pages in a content portfolio should a human reviewer prioritize for refresh, and does a learned model rank them better than a simple, auditable rule? I built a transparent baseline using staleness, search demand, ranking position, and CTR, then compared it with Logistic Regression on the same anonymized FlyRank content dataset. The evaluation used a client-grouped 80/20 split, with 25 clients and 23,837 rows for training and 7 unseen clients and 6,163 rows for testing, with zero client overlap. On the held-out test set, Logistic Regression achieved Precision@50 of **0.72**, compared with **0.54** for the Week-4 baseline, an observed **18-percentage-point improvement** on this split. The resulting system is best treated as a ranked decision-support queue for human review, not as a causal model or a guarantee that refreshing a page will improve performance.

## 1. Introduction / Problem Statement

Content teams often have more pages that could be reviewed than they have time to review. The practical decision is therefore:

> **Which pages should an editor or SEO analyst review first for refresh, optimisation, or continued monitoring?**

The unit of analysis is an individual webpage represented by `content_id`.

The desired output is not simply a probability. It is a ranked queue that helps a human decide what to inspect first and why.

A simple rule is useful because it is transparent and easy to audit. However, content performance can reflect several signals at once, including historical engagement, freshness, search demand, ranking position, content format, and intent. Logistic Regression provides an interpretable first model that can combine these signals and be compared fairly against the hand-written baseline.

This project therefore belongs to the **Refresh / Content Opportunity Scoring** lane.

## 2. Data

### Dataset

The analysis uses the FlyRank ML Internship Search Intelligence capstone release:

- **Table:** `content_refresh_anonymized.csv`
- **Rows:** 30,000
- **Columns:** 44
- **Pseudonymous clients:** 32
- **Unit:** content item / webpage

### Reporting window

The anonymized release does not expose calendar start/end dates. The available performance fields describe a **90-day performance window**, while the trend fields compare the **last 30 days with the previous 30 days**.

This distinction matters: the model is evaluated using the observed trend field, while the input features are restricted to information treated as available from the historical content state.

### Public-safe exclusions

The release and this paper exclude:

- client names
- domains
- private search queries
- raw URLs
- credentials
- other client-identifying information

The dataset uses pseudonymous `client_id` and `content_id` values.

## 3. Methodology

### 3.1 Baseline

The Week-4 baseline is a transparent scoring rule.

A page receives more priority when it:

1. has not been updated for a long time;
2. has meaningful search demand;
3. ranks around positions 8–20, where improvement may be actionable;
4. has low CTR.

The baseline produces reason codes that connect signals to actions:

| Reason code | Meaning | Recommended action |
|---|---|---|
| `STALE_REFRESH` | Old page with a refresh opportunity | Refresh Content |
| `CTR_FIX` | CTR opportunity | Improve Title & Meta |
| `QUICK_WIN` | Near page one with meaningful demand | Optimise Existing Content |
| `LOW_PRIORITY` | Does not meet the main signals | Monitor |

The baseline is intentionally simple. Its purpose is to create an auditable benchmark rather than to claim that these thresholds are universally optimal.

### 3.2 Model

I selected **Logistic Regression** as the first machine-learning method.

It fits the task because:

- it produces a probability score that can rank pages;
- its coefficients can be inspected;
- it is relatively simple to reproduce;
- it provides a fair first comparison against the transparent baseline.

The observed target is:

```text
target = 1 when trend_direction == "down"
target = 0 otherwise
```

This is an observed historical outcome. It should not be described as a guaranteed future prediction.

### 3.3 Split design

I used `GroupShuffleSplit` with `client_id` as the grouping variable and `random_state = 42`.

| Split | Rows | Clients |
|---|---:|---:|
| Train | 23,837 | 25 |
| Test | 6,163 | 7 |

**Client overlap: 0**

Grouping by client is more conservative than randomly splitting rows because pages belonging to the same client can share characteristics. A random row split could therefore make evaluation look easier by placing related pages in both training and testing.

The observed decline rate was **55.0%** in training and **51.1%** in testing.

### 3.4 Leakage controls

The following fields were excluded from model inputs:

- `trend_direction`
- `trend_pct`
- `impressions_last_30d`
- `impressions_prev_30d`
- baseline-generated `action_label`
- baseline-generated `reason_code`
- baseline-generated `score`

The first two define the target. The last-30-day impression fields directly contribute to the observed trend calculation. The baseline outputs are excluded because they are decisions produced by the benchmark rather than independent real-world inputs.

The remaining features describe historical content, search, traffic, engagement, and content attributes.

## 4. Results

### 4.1 Model versus baseline

Both approaches were evaluated on the **same held-out clients**, using **Precision@50**.

| Method | Precision@50 |
|---|---:|
| Week-4 rule-based baseline | **0.54** |
| Logistic Regression | **0.72** |
| Test base rate | **0.511** |

The Logistic Regression model therefore produced an observed **18 percentage-point improvement** in Precision@50 over the baseline on this particular held-out client split.

This means that, among the 50 highest-ranked test items, the model selected more items matching the observed decline label than the baseline did.

It does **not** mean that the model will always achieve 72% precision on new data, nor that refreshing those pages will cause recovery.

### 4.2 Error analysis

The model had **14 false positives in its top-50 predictions**.

A false positive here means that an item was ranked in the model's top 50 but its observed test target was not `down`.

This is important because the system is intended for human decision support. A high score should mean:

> **Review this page first.**

It should not mean:

> **This page definitely needs a refresh.**

### 4.3 What the model relied on

The largest absolute Logistic Regression coefficients included:

| Feature | Coefficient |
|---|---:|
| `users_90d` | -0.988 |
| `sessions_90d` | +0.770 |
| `days_with_impressions` | +0.604 |
| `main_intent_navigational` | -0.450 |
| `days_with_sessions` | -0.426 |
| `content_type_keyword article` | +0.357 |
| `content_type_feedly article` | -0.339 |
| `content_age_days` | -0.289 |
| `scroll_events_90d` | +0.281 |
| `word_count` | +0.242 |

These coefficients describe **directional associations within this fitted model**. They are not causal effects.

The model's strongest signals were largely related to historical engagement and consistency. This suggests that age or search volume alone should not be treated as sufficient evidence for a refresh decision.

## 5. Validation and Research Context

The weekly validation work also compared the modeling choices with findings from the FlyRank research material.

Two relevant findings were:

1. Growing content in the broader FlyRank research dataset tended to be younger and longer than declining content.
2. Content performance showed a lifecycle pattern in which performance weakened in older age bands, while refreshed older pages could behave differently.

These findings provide useful context for including freshness and content-depth variables, but they do not prove that age or word count causes decline.

The capstone therefore treats these signals as evidence to investigate rather than as automatic rules.

## 6. Limitations & Honest Framing

This project should be interpreted as **decision-support evidence**.

### Observed, not causal

The reported 0.72 Precision@50 is an observed result on one held-out client split. The coefficients show associations with the observed decline label; they do not establish causal mechanisms.

### One split and one seed

The evaluation uses one client-grouped split with `random_state = 42`. A different held-out set could produce a different result.

### Directional label

The target is derived from `trend_direction`. It is therefore an operational definition of decline rather than an independent ground-truth measure of business harm.

### Portfolio-specific

The dataset contains 30,000 anonymized records across 32 clients. Results may not generalise to a different portfolio, industry mix, traffic distribution, or content system.

### Priority is not success probability

A high-ranked page is a page worth reviewing. It is not proof that a refresh will work.

### No causal Google claim

Nothing in this analysis establishes how Google's ranking system works, proves an algorithmic cause, or demonstrates that a particular content change causes a ranking recovery.

## 7. Ranked Recommendations

The practical output is a ranked, reason-coded review queue.

### Priority 1 — Stale content with strong opportunity signals

**Reason code:** `STALE_REFRESH`  
**Action:** Refresh Content

Prioritise pages that combine long time since update with meaningful visibility or demand signals. Review whether the content has become outdated, thin, incomplete, or less aligned with current search intent.

### Priority 2 — Near-page-one opportunities

**Reason code:** `QUICK_WIN`  
**Action:** Optimise Existing Content

Review pages ranking around positions 8–20 with meaningful search demand. These are candidates for targeted optimisation rather than automatic rewriting.

### Priority 3 — CTR opportunities

**Reason code:** `CTR_FIX`  
**Action:** Improve Title & Meta

Investigate low CTR in context. A low CTR does not automatically mean the metadata is wrong; SERP features, intent mismatch, and other factors can contribute.

### Priority 4 — Weak or ambiguous signals

**Reason code:** `LOW_PRIORITY` / `MONITOR`  
**Action:** Monitor

Avoid spending immediate editorial effort on pages without a clear opportunity signal. Continue monitoring until stronger evidence appears.

### Action principle

The model and baseline should be used as a **review queue**, with a human checking the page before taking action.

## 8. Reproducibility

The capstone is designed to be reproducible from the repository.

Expected notebook structure:

```text
work/
└── notebooks/
    ├── w04_baseline_score.ipynb
    ├── w05_model.ipynb
    ├── w06_validation_audit.ipynb
    └── w07_action_playbook.ipynb
```

The main modeling run uses:

```text
random_state = 42
```

Core Python packages include:

```text
pandas
numpy
scikit-learn
```

A pinned `requirements.txt` should be included at the repository root before final submission.

The baseline/action queue is exported as a CSV so that the ranked output can be inspected independently of the notebook.

The deployed paper URL belongs in:

```text
submission/paper_url.txt
```

That file should contain exactly one line: the direct URL of the deployed paper.

## 9. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset**.

Data source: [FlyRank](https://flyrank.ai)

This project uses the anonymized capstone release for research and educational analysis. No client names, domains, private queries, credentials, or other identifying information are published.

---

## Final takeaway

The main result is straightforward:

> **On the held-out client split used here, Logistic Regression ranked declining content more precisely than the Week-4 hand-written baseline at the top of the queue: 0.72 versus 0.54 Precision@50.**

The important qualification is equally straightforward:

> **This is an observed, portfolio-specific decision-support result — not a causal claim, not a guarantee of future performance, and not proof of how a search engine ranks content.**
