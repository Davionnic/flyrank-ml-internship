# Content Action Playbook — Refresh / Content Opportunity Scoring

Decision-support ranking for which page a reviewer should open first.
Not causal SEO; not an auto-publish system.

## Ranking quality (client-holdout)

- Best model: `random_forest`
- Precision@50: **0.680** (baseline **0.240**, holdout base rate **0.391**)
- Precision@20: **0.700** (baseline **0.150**)
- Holdout top-50 proxy positives: **34/50**

## Suggested actions

- `monitor`: 13,264
- `refresh`: 8,040
- `refresh_and_review_ctr`: 6,646
- `refresh_and_review_engagement`: 1,968
- `expand_and_refresh`: 82

## Confidence mix

- `low`: 15,000
- `medium`: 11,368
- `high`: 3,632

## Top reason codes

- `declining_with_demand`: 13,152
- `visible_model_opportunity`: 10,590
- `low_ctr_visible_page`: 9,759
- `ctr_review_candidate`: 9,759
- `general_refresh_review`: 9,117
- `model_decline_risk`: 7,927
- `page_one_decay_risk`: 7,076
- `low_engagement_visible_page`: 6,508
- `engagement_review_candidate`: 6,508
- `thin_visible_page`: 82

## Human review (short)

Verify live page, intent, seasonality, duplicates, and brand/legal before acting.
Never auto-publish from the score alone.

## Monitoring triggers

- Prec@50 approaches baseline or base rate → retrain / pause automation.
- Action-mix shock or high editor-reject rate → recalibrate + sample review.
- Label or feature-schema change → new contract + full rebuild.

## Claim stance

observed / measured / directional / decision-support ranking.
