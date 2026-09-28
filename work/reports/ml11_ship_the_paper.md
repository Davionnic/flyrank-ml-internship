# Ranking Pages for Refresh Review: A Client-Holdout Decision-Support Model

- **Author:** Dave Andrei Almia Gallo (`Davionnic`)
- **Assignment:** ML-11 — Ship the Paper (Week 8 weekly)
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/Davionnic/flyrank-ml-internship
- **Deployed paper:** see `submission/paper_url.txt`
- **Claim stance:** observed / measured / directional / **decision-support** (not causal)

> This short paper ships the Refresh / Content Opportunity Scoring lane work from ML-07–ML-10.
> Capstone notebook `work/notebooks/capstone.ipynb` is intentionally untouched for this weekly card.

---

## Abstract

Which existing pages should a content reviewer open first when organic demand is slipping?
Using FlyRank’s anonymized content-refresh starter set (30,000 pages × 44 columns), we frame a
ranking task whose proxy label is `is_declining_label = (trend_direction == "down")`.
A transparent hand-rule baseline reaches Precision@50 = **0.240** on a client-holdout split;
a Random Forest trained without label-derived leak features reaches Precision@50 = **0.680**
(~2.8× the baseline; holdout base rate 0.391). The output is a decision-support queue with
reason codes and suggested actions — not a causal claim about SEO interventions.

## 1. Introduction / Problem

Content teams cannot re-audit every URL every week. The practical decision is ranked triage:
given limited editor time, which pages are most worth a human refresh review *right now*?

Unit of analysis is a page. The model outputs a score, a suggested action
(`monitor` / `refresh` / `refresh_and_review_ctr` / …), confidence, and reason codes.
A wrong high rank wastes editor time; a missed declining high-demand page leaves recoverable
traffic on the table. ML helps when a transparent rule undershoots and a validated ranker
concentrates true decline proxies in the top of the queue — on a split that does not leak
client identity.

## 2. Data

| Item | Detail |
|---|---|
| Source | FlyRank ML Internship anonymized starter (`data/raw/content_refresh_anonymized.csv`) |
| Size | 30,000 pages × 44 public-safe columns |
| Label | `is_declining_label = (trend_direction == "down")` — full-set base rate ≈ 0.542 |
| Excluded from features | `trend_direction`, `trend_pct`, and any client-/URL-/query-identifying fields |
| Public-safety | No private names, domains as identifiers, addresses, or raw queries in this paper |

Receipts: `work/outputs/model_holdout_metrics.json`, `validation_audit_metrics.json`,
`baseline_action_score_metrics.json`, `action_playbook_metrics.json`.

## 3. Methodology

**Assumptions.** Cross-sectional snapshot; decline label is a *proxy* for refresh worthiness,
not ground-truth editor judgment. Rankings support review, not auto-publish.

**Baseline (ML-07).** Weighted hand score:
`0.4·visibility + 0.3·freshness_risk + 0.25·position_opportunity + 0.05·depth_gap`.
Client-holdout Precision@50 = **0.240** (Prec@20 = 0.150).

**Models (ML-08).** Logistic regression, decision tree, Random Forest on 52 engineered
numeric/categorical features (seed 42). Same client-holdout: 6 held-out clients,
27,675 train / 2,325 test rows; holdout base rate **0.391**.

**Leakage controls (ML-09).** Forbidden overlap with label fields is empty in the feature list.
Injecting `trend_pct` (or last/prev 30d leak proxies) collapses Prec@50 to 1.0 — a deliberate
red-team showing why those columns stay out. Random row split Prec@50 = 0.90 vs client-holdout
0.68 (gap ≈ 0.22) — the honest number is the grouped split.

**Playbook blend (ML-10).** Final queue score ≈ `0.7·model_probability + 0.3·baseline_normalized`,
plus reason codes and human-review gates.

## 4. Results

### Model vs baseline (same client-holdout)

| Method | Prec@20 | Prec@50 | Avg precision | ROC AUC |
|---|---:|---:|---:|---:|
| Baseline rules | 0.150 | **0.240** | — | — |
| Logistic regression | 0.350 | 0.400 | 0.522 | 0.700 |
| Decision tree | 0.650 | 0.660 | 0.575 | 0.742 |
| **Random Forest (selected)** | **0.700** | **0.680** | **0.610** | **0.747** |

Holdout top-50 proxy positives: **34 / 50**. Headline lift: **0.240 → 0.680** (≈2.8×) at K=50,
above the holdout base rate of 0.391.

### Split audit

| Split | RF Prec@50 |
|---|---:|
| Random stratified row | 0.900 |
| Client holdout (reported) | **0.680** |

### Playbook action mix (30,000 pages scored)

| Action | Count |
|---|---:|
| monitor | 13,264 |
| refresh | 8,040 |
| refresh_and_review_ctr | 6,646 |
| refresh_and_review_engagement | 1,968 |
| expand_and_refresh | 82 |

Confidence mix: low 15,000 / medium 11,368 / high 3,632.

Top reason codes include `declining_with_demand`, `visible_model_opportunity`,
`low_ctr_visible_page`, and `model_decline_risk`.

Top RF signals (importance): `log_impressions_90d`, `days_with_impressions`, `avg_position`,
`content_age_days`, then depth/CTR/engagement features — demand and visibility dominate,
which matches a triage story rather than a “secret ranking factor” story.

## 5. Limitations & honest framing

- **Decision-support, not causal.** This study does not measure whether refreshing a page
  *causes* recovery. We observe associations and out-of-sample ranking quality only.
- **Proxy label.** `trend_direction == "down"` is not editor intent; false positives/negatives
  relative to human judgment are expected.
- **Single snapshot / starter sample.** Not a claim about Google’s algorithm or all sites.
- **Client holdout ≠ time travel.** Grouped split blocks identity leakage; it does not fully
  substitute for a sealed future month.
- **Base rate matters.** Prec@50 = 0.68 must be read next to holdout base rate 0.391 and
  baseline 0.240 — not as standalone accuracy.

## 6. Ranked recommendations (so what)

1. Open the top of the blended queue first (`work/outputs/action_playbook_top20.md`).
2. Treat high-confidence `refresh_and_review_ctr` rows as “verify live CTR/snippet before
   rewriting,” not auto-publish.
3. Keep `monitor` as the default mass action — most pages should not enter a rewrite sprint.
4. Pause / retrain if Prec@50 drifts toward baseline or base rate, or if editor reject-rate spikes.
5. Never ship causal SEO language from these numbers.

## 7. Reproducibility

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python scripts/run_all.py   # reference pipeline
# Lane receipts already committed under work/outputs/*.json (seeds: random_state=42)
```

Primary notebooks (prior weeks): `work/notebooks/w04_baseline_score.ipynb`,
`w05_model.ipynb`, `w06_validation_audit.ipynb`, `w07_action_playbook.ipynb`.
sklearn 1.9.1. Metrics JSON files are the receipts this paper’s numbers trace to.

## 8. Acknowledgments & data credit

Built on the **FlyRank ML Internship** dataset — [https://flyrank.ai](https://flyrank.ai).
Reference pipeline and anonymized starter ship with the public internship repo.
