# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Dave Andrei Almia Gallo (`Davionnic`) · gallodave.cs@gmail.com
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/Davionnic/flyrank-ml-internship
- **Date:** 2026-10-01 (Asia/Shanghai)
- **Deployed paper:** https://davionnic.github.io/flyrank-ml-internship/paper/
- **Claim stance:** observed / measured / directional / **decision-support** (not causal)

> Filled from `work/capstone_report_template.md`. Numbers below are receipts from committed
> `work/outputs/*.json` (same values as a fresh load of those files). Framing: decision-support
> ranked review queue — not causal SEO.

## 0. Abstract

Which existing pages should a content reviewer open first when organic demand looks soft?
Using FlyRank’s anonymized content-refresh starter set (30,000 pages × 44 columns), we frame a
ranking task whose proxy label is `is_declining_label = (trend_direction == "down")`.
A transparent hand-rule baseline reaches Precision@50 = **0.240** on a client-holdout split;
a Random Forest trained without label-derived leak features reaches Precision@50 = **0.680**
(~2.8× the baseline; holdout base rate 0.391). The output is a decision-support queue with
reason codes and suggested actions — not a causal claim about SEO interventions.

## 1. Problem framing

**Decision supported:** ranked triage for refresh review. Editors cannot re-audit every URL
every week; they need an ordered queue of which pages to open first when demand looks soft.

| Axis | Choice |
|---|---|
| Unit of analysis | One page (content item) |
| Output | Score + suggested action + confidence + reason codes |
| Human action | Open / refresh / check CTR or engagement / keep monitoring |
| Cost of a wrong high rank | Wasted editor time on a page that did not need review |
| Cost of a miss | A high-demand declining page stays unreviewed |
| Why ML | A transparent hand rule undershoots; a validated ranker concentrates proxy declines at the top of the queue on a split that does not leak client identity |

This is **decision-support ranking**, not auto-publish and not a claim that refreshing causes
traffic recovery.

## 2. Data safety

**Used:** FlyRank ML Internship anonymized starter
`data/raw/content_refresh_anonymized.csv` — 30,000 rows × 44 public-safe columns, 32
pseudonymous clients, trailing-90-day metrics.

**Proxy label:** `is_declining_label = (trend_direction == "down")`. Full-set base rate ≈
**0.542**; client-holdout test base rate ≈ **0.391**
(`work/outputs/model_holdout_metrics.json`).

**Deliberately excluded from features:**

- Label-derived / leak fields: `trend_direction`, `trend_pct`, and the label itself
- Identifying / raw text: client names, live URLs as features, private query strings, titles
  as free text features
- Pseudonymous IDs (`client_id`, `content_id`): grouping and split only — **never** model
  features

**Leakage risks considered (ML-05 / ML-09):**

- Forbidden overlap with label fields in the feature list is **empty**
- Red-team: injecting `trend_pct` (or last/prev 30d leak proxies) collapses Prec@50 to
  **1.0** — those columns stay out
- Random row split inflates RF Prec@50 to **0.90** vs client-holdout **0.68** (gap ≈ 0.22);
  the published number is the grouped split

**Public-safety check:** no client names, live domains as identifiers, addresses, or raw
queries appear in `work/` reports, notebooks, or the deployed paper. Metrics JSONs use
pseudonymous `client_*` ids for the holdout list only.

## 3. Baseline

Transparent weighted hand score (ML-07), same client-holdout and same Precision@K metric as
the models:

```text
0.4·visibility + 0.3·freshness_risk + 0.25·position_opportunity + 0.05·depth_gap
```

| Metric (client-holdout) | Value |
|---|---:|
| Precision@10 | 0.20 |
| Precision@20 | 0.15 |
| **Precision@50** | **0.24** |
| Holdout base rate | 0.391 |

Receipt: `work/outputs/baseline_action_score_metrics.json`. Fair comparison because the
baseline uses the identical split, label, and K as the model evaluation.

## 4. Model / analysis

**Method:** Logistic regression (scaled, class-balanced), decision tree (depth 5), Random
Forest (depth 10, 200 trees) — same family as `scripts/03_train_model.py`. Selected by
Precision@50 on client-holdout: **random_forest**.

**Why it fits the lane:** The product question is “which page first?”, so we rank by
`predict_proba` and score Precision@K — not threshold accuracy alone.

**Features:** 52 engineered numeric / one-hot categorical features after excluding leak and
ID columns (`feature_count` in `model_holdout_metrics.json`). Demand/visibility, position,
age, depth, CTR, and engagement texture dominate importance.

**Target / proxy (one sentence):** `is_declining_label` is true when
`trend_direction == "down"` — a teaching proxy for “worth a refresh look,” not editor ground
truth.

**Playbook blend (ML-10):** final queue score ≈
`0.7·model_probability + 0.3·baseline_normalized`, plus reason codes and human-review gates.

## 5. Evaluation

**Split:** client-holdout, `random_state=42`, 6 held-out clients, **27,675** train /
**2,325** test rows. Why: pages from the same client share niche and CMS habits; a random
row split leaks that style. Client-holdout asks whether the score helps on accounts the
model never saw. The starter CSV is a same-window teaching slice, so this is not a sealed
future-month design.

**Same-split comparison (primary metric Precision@50):**

| Method | Prec@20 | Prec@50 | Avg precision | ROC AUC |
|---|---:|---:|---:|---:|
| Baseline rules | 0.150 | **0.240** | — | — |
| Logistic regression | 0.350 | 0.400 | 0.522 | 0.700 |
| Decision tree | 0.650 | 0.660 | 0.575 | 0.742 |
| **Random Forest (selected)** | **0.700** | **0.680** | **0.610** | **0.747** |

Holdout top-50 proxy positives: **34 / 50**. Headline: **0.240 → 0.680** (≈2.8×) at K=50,
above holdout base rate **0.391**.

**Split audit:** random stratified row RF Prec@50 = **0.90** vs client-holdout **0.68** —
we publish the harder number (`validation_audit_metrics.json`).

**Short error read:** The model is right when high-demand / soft-position / aged pages with
decline proxies rise to the top. It is wrong when high score meets a proxy-stable page
(false open) or a soft-demand true decline sits lower (miss). Editor judgment remains the
gate.

Receipts: `work/outputs/model_holdout_metrics.json`, `validation_audit_metrics.json`.

## 6. Interpretation

**What it found:** Demand and visibility features lead Random Forest importance —
`log_impressions_90d`, `days_with_impressions`, `avg_position`, `content_age_days`, then
depth / CTR / engagement. That matches a triage story (“where is there demand that looks
soft?”), not a “secret Google ranking factor” story.

**Surprises / negatives:**

- Hand baseline Prec@50 (0.24) sits *below* holdout base rate (0.391) — the obvious rule is
  not a free win
- Random-split optimism is large (+0.22 Prec@50); publishing only that number would be
  dishonest
- Leak injection hitting 1.0 is a successful red-team, not a model to ship

**Playbook mix (30,000 pages scored):** monitor 13,264 · refresh 8,040 ·
refresh_and_review_ctr 6,646 · refresh_and_review_engagement 1,968 · expand_and_refresh 82.
Confidence: low 15,000 / medium 11,368 / high 3,632. Top reason codes include
`declining_with_demand`, `visible_model_opportunity`, `low_ctr_visible_page`,
`model_decline_risk`.

## 7. Recommendation

How a FlyRank editor would use this tomorrow:

1. Open the top of the blended queue first (`work/outputs/action_playbook_top20.md`).
2. Treat high-confidence `refresh_and_review_ctr` as “verify live CTR/snippet before
   rewriting,” not auto-publish.
3. Keep `monitor` as the default mass action — most pages should not enter a rewrite sprint.
4. Pause or retrain if Prec@50 drifts toward baseline/base rate, or if editor reject-rate
   spikes.
5. Never ship causal SEO language from these numbers.

**Confidence:** Medium-high on *ranking quality vs the hand rule on this holdout*; low on
any claim that refresh *causes* recovery. Limits: proxy label, snapshot starter sample,
client-holdout ≠ sealed future month.

## 8. Reproducibility

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python scripts/run_all.py
# Capstone notebook (loads committed receipts; optional live CSV checks):
# jupyter nbconvert --execute --to notebook --inplace work/notebooks/capstone.ipynb
```

| Item | Value |
|---|---|
| Seeds | `random_state=42` |
| sklearn | 1.9.1 (matches committed metrics JSON) |
| Sealed-holdout builder + metrics | Lane notebooks `w04`–`w07` + `work/outputs/*.json` |
| Capstone notebook | `work/notebooks/capstone.ipynb` |
| Paper twin | `work/reports/ml11_ship_the_paper.md` → `docs/paper/` |
| Paper URL file | `submission/paper_url.txt` (one line) |

Environment: see root `requirements.txt` (pandas, numpy, scikit-learn, matplotlib,
reportlab).

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship** dataset — [https://flyrank.ai](https://flyrank.ai).
The reference pipeline and anonymized starter ship with the public internship repository.
Author: Dave Andrei Almia Gallo (`Davionnic`).

---

> **Claims checklist:** observed / measured / directional / decision-support language
> throughout · no causal claims · no “predicted Google’s algorithm” · no client-identifying
> details · Prec@50 reported next to holdout base rate 0.391 and baseline 0.240 · numbers
> match committed `work/outputs/*.json` loaded by `work/notebooks/capstone.ipynb`.
