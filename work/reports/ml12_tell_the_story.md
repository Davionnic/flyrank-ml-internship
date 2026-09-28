# ML-12 — Tell the Story

- **Author:** Dave Andrei Almia Gallo (`Davionnic`)
- **Assignment:** ML-12 — Tell the Story (Week 8 weekly)
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/Davionnic/flyrank-ml-internship
- **Paper:** https://davionnic.github.io/flyrank-ml-internship/paper/
- **Paper source:** `work/reports/ml11_ship_the_paper.md`
- **Claim stance:** observed / measured / directional / **decision-support** (not causal)

> Required pieces from `work/README.md`: 5-minute demo outline + social-post cut +
> 3-sentence employer-facing summary. Repo does **not** require a video. Capstone notebook
> `work/notebooks/capstone.ipynb` is intentionally untouched (same rule as ML-11).

Metrics below are receipts only — baseline Prec@50 **0.240**, Random Forest Prec@50 **0.680**
on client-holdout (`work/outputs/*.json`, sklearn 1.9.1, `random_state=42`).

---

## 1. Five-minute demo outline

**Audience:** mentor / peer review / hiring screen.  
**Goal:** show the decision, the split, the lift, and the limit — then stop.

| Min | Beat | What you say / show |
|---:|---|---|
| 0:00–0:40 | The decision | Content teams cannot re-audit 30k pages. The job is ranked triage: which URLs open first when organic demand looks soft? |
| 0:40–1:20 | The frame | Page-level ranking. Proxy label: `is_declining_label = (trend_direction == "down")`. Output is a review queue with reason codes and suggested actions — not “SEO magic.” |
| 1:20–2:10 | Why the obvious rule fails | Hand-rule baseline (visibility + freshness + position + depth). Same client-holdout: Prec@50 = **0.240**. Too many false opens at the top of the queue. |
| 2:10–3:20 | What changed | Random Forest on 52 engineered features, **no** `trend_direction` / `trend_pct` (or other label leaks). Client holdout: 6 clients out, 27,675 / 2,325 rows. Prec@50 = **0.680** (34/50 proxy positives). Lift ≈ **2.8×** over baseline; holdout base rate 0.391. |
| 3:20–4:10 | Honest split + playbook | Random row split would claim Prec@50 = 0.90 — that number is inflated. Client-holdout 0.680 is the one that ships. Blended queue (~0.7 model + ~0.3 baseline) → actions (`monitor` / `refresh` / CTR review / …) with reason codes. |
| 4:10–4:50 | Limits (say them yourself) | Decision-support ranking, **not** causal. Proxy label ≠ editor judgment. Snapshot starter data. Does not predict Google’s algorithm. Open the paper: one URL. |
| 4:50–5:00 | Close | Ask: “Where would you stress-test this next — time-split holdout or editor accept/reject logging?” |

**Props (optional, keep to two):** paper page + `work/outputs/action_playbook_top20.md` (or the Prec@50 table from the paper). No slide deck required.

**Do not say:** “causes traffic,” “beats Google,” “proves refresh works.”

---

## 2. Social-post cut (LinkedIn-style)

Copy-paste ready. Plain voice. One finding, one method line, one link.

---

I spent this internship on one boring, useful question:

**Which existing pages should a reviewer open first when demand looks like it’s slipping?**

Data: FlyRank’s anonymized content-refresh set — 30,000 pages × 44 columns.

Method: client-holdout ranking (no client in both train and test). Transparent hand rules vs a Random Forest. Label is a decline **proxy**, not editor ground truth. Features exclude the label fields on purpose.

Result on that holdout:
- Baseline Prec@50 = **0.240**
- Random Forest Prec@50 = **0.680** (~2.8×)
- Holdout base rate = 0.391

That’s a decision-support queue with reason codes — not a causal SEO claim.

Paper (full limits + receipts):  
https://davionnic.github.io/flyrank-ml-internship/paper/

Repo: https://github.com/Davionnic/flyrank-ml-internship

#MachineLearning #SEO #DecisionSupport #FlyRank

---

**Shorter alt (if character-capped):**

Built a client-holdout refresh ranking model on 30k anonymized pages. Hand rules hit Prec@50 0.240; Random Forest hit 0.680. Decision-support triage, not causal SEO. Paper: https://davionnic.github.io/flyrank-ml-internship/paper/

---

## 3. Employer-facing summary (3 sentences)

I built a page-level **refresh / content opportunity** ranker that turns an anonymized 30,000-page FlyRank starter set into a review queue with reason codes and suggested actions. On a **client-holdout** split (train/test share no clients), a Random Forest reaches Precision@50 = **0.680** versus a transparent rule baseline at **0.240**, with the decline label treated as a proxy and leak fields kept out of the feature set. The system is **decision-support ranking** — it tells editors what to open first; it does not claim that refreshing a URL causes recovery.

---

## 4. Artifact map (for reviewers)

| Piece | Path / URL |
|---|---|
| This ML-12 story pack | `work/reports/ml12_tell_the_story.md` |
| Shipped paper (ML-11) | `work/reports/ml11_ship_the_paper.md` |
| Deployed paper | https://davionnic.github.io/flyrank-ml-internship/paper/ |
| Metrics receipts | `work/outputs/baseline_action_score_metrics.json`, `model_holdout_metrics.json`, `validation_audit_metrics.json`, `action_playbook_metrics.json` |
| Capstone notebook | untouched (not part of this weekly card) |

**Video:** not required by this repo’s ML-12 definition. No video artifact produced; text pack above is the deliverable.

**Portal:** not submitted from this commit (repo-only, per assignment instructions for this run).
