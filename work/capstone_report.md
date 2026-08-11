# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Eman Hrustemović
- **Lane:** Lane 2 — Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/EmanHrustemovic/FlyRank-AI-Intership-ML-Track-
- **Date:** August 2026

---

## 1. Problem framing

This work supports a content-team decision: **which pages should be prioritized for
refresh, given limited editorial time?** The unit of analysis is one **page**,
aggregated from daily FlyRank warehouse data within a single month (`month=2026-03`).
The output is a **ranked queue with a reason code per page** (e.g. `WEAK_POSITION`,
`STALE_CONTENT`), plus a binary score approximating whether the page is likely
declining in search position. The action a human takes is manual review and refresh
of the highest-ranked pages — not automated publishing. The cost of a wrong call is
asymmetric: a false positive wastes editorial time on a page that wasn't really
declining; a false negative lets a genuinely declining page keep losing visibility
unnoticed. ML helps here because the underlying signals (position, CTR, staleness,
volume) interact — no single hand-written rule captured the pattern as well as a
model that could weigh them jointly (see Section 3 vs. Section 4).

## 2. Data safety

**Data used:** `fact_content_daily_performance` (`month=2026-03`, ~9.84M rows) and
`dim_content`, from the FlyRank internship Hugging Face warehouse release.

**Deliberately excluded:**
- Rows where `gsc_data_available` is not `TRUE` — only ~37% of rows have usable GSC
  data; the rest are excluded rather than imputed.
- `avg_position_late` and any other late-window (post-decision-point) fields — these
  are label-derived. A deliberate leakage test (ML-04) added `avg_position_late` as a
  feature and watched AUC jump from ~0.68 to 0.9999, then removed it.
- `client_hash_id` is used **only for grouping** in the validation split (Section 5),
  never as a model feature.
- The `_sample` file (June 2026, the final month in the warehouse) was used only to
  test query mechanics early on, never for label or feature development, since it is
  the natural outcome window for any past→future label.

**Leakage risk considered:** the label (`is_declining_label`) is derived by comparing
the first half of March to the second half. All features are computed **only** from
the first half (`impr_early`, `avg_position_early`, `ctr_early`) plus static content
metadata (`word_count`, `days_since_update`), so no feature crosses into the label's
outcome window. This was independently re-audited in ML-09 (Section 5 below).

No client names, domains, URLs, or private queries appear anywhere in `work/` — all
identifiers are salted hashes provided by the warehouse release itself.

## 3. Baseline

The baseline is a single-signal heuristic: **flag a page if its observed CTR is
significantly below the expected CTR for its position bucket** (bucket-average CTR
computed from the same data). This is a fair comparison because it's evaluated on
the exact same test rows and the exact same metric (Precision@50) as the model below.

**Baseline result:** Precision@50 = **0.700**, AUC = **0.544**.

Before building this rule, two candidate signals were tested directly (ML-07):
- **Staleness** (`days_since_update`) — **OPPOSITE** of expected: the freshest bucket
  (0–30 days) had the *highest* decline rate (54.1%, n=140,458), not the lowest.
  Dropped as a rule driver.
- **CTR-vs-position** — **CONFIRMED**: a clean, monotonic drop from 0.346% CTR (top 3
  positions) to 0.128% CTR (position 20+), n ranging 10,893–53,068 per bucket. Used as
  the baseline's foundation.

## 4. Model / analysis

**Method:** Logistic Regression, chosen for interpretability and because its
probability outputs rank cleanly for a Precision@K task. Random Forest was trained
as a comparison point (Section 5).

**Feature list:** `impr_early`, `avg_position_early`, `ctr_early`, `word_count`,
`days_since_update` — all computable at the decision point (mid-month), none derived
from the outcome window. `avg_position_late` was deliberately left out (see Section 2).

**Target:** `is_declining_label` — 1 if a page's average search position in the
second half of March is worse (numerically higher) than in the first half, else 0.
**Base rate:** 52.0% positive class (roughly balanced by construction, not by design
choice — it reflects the real split in this month's data).

## 5. Evaluation

**Split:** two splits were compared. The initial split (ML-08) was a random 70/30
stratified split. Because pages from the same client could appear in both train and
test under a random split, ML-09 re-ran the same model under a **client-grouped
split** (`GroupShuffleSplit`, confirmed 0 client overlap between train and test) —
the more honest evaluation for this task, since the goal is generalizing across
clients, not memorizing client-specific patterns.

| Method | Precision@50 | AUC | Split |
|---|---|---|---|
| Baseline (CTR rule) | 0.700 | 0.544 | random |
| Logistic Regression | 0.960 | 0.675 | random |
| Random Forest | 0.920 | 0.704 | random |
| **Logistic Regression** | **0.960** | **0.649** | **client-grouped (honest)** |

Precision@50 held steady (0.960) under the stricter split — the top-50 ranked
predictions don't depend on client-specific leakage. AUC dropped modestly
(0.675 → 0.649), suggesting a small amount of the original AUC was inflated by the
model learning client-level patterns under the random split. Base rate is 52.0%, so
Precision@50 = 0.96 is a substantial lift over chance-level precision at that rate.

**Error analysis:** in the top-50 ranked list, only 2 of 50 were false positives.
Both shared an identical, anomalous profile: `avg_position_early` near 0 (GSC
positions are 1-indexed, so this is likely a data aggregation artifact) and a
*negative* `days_since_update` (implying an update date after the decision point —
a data quality issue, not a real feature value). This is a known, named limitation,
not a silent failure.

## 6. Interpretation

Logistic Regression coefficients (unscaled, so magnitudes reflect both effect size
and feature scale):

- `avg_position_early` (**-0.048**, strongest): pages already ranking poorly show a
  *lower* predicted decline probability — a floor effect. A page already near the
  bottom has less room to fall further.
- `days_since_update` (**-0.0018**): more days since update is associated with
  *lower* decline probability — consistent with the OPPOSITE finding in Section 3.
  Staleness alone does not predict decline in this data; the model learned the same
  negative relationship independently.
- `ctr_early` (**+0.009**): weakly positive, but given `ctr_early`'s tiny scale
  (~0.0001–0.003), its real-world influence is minor relative to position.
- `impr_early` and `word_count`: near-zero coefficients, negligible influence.

**Surprise / negative result worth keeping:** staleness was hypothesized as an
intuitive decline driver going in. It wasn't. Reporting this OPPOSITE finding, rather
than quietly dropping it, is itself a valid and useful result — it prevented the
final rule and model from being built on a false premise.

## 7. Recommendation

The ML-10 playbook scores all 92,148 usable pages and assigns each a reason code
based on which standardized feature contributed most to its predicted decline
probability:

| Reason code | Pages | Share | Suggested action |
|---|---|---|---|
| `WEAK_POSITION` | 63,197 | 68.7% | improve-ranking |
| `STALE_CONTENT` | 13,508 | 14.7% | refresh |
| `THIN_CONTENT` | 8,276 | 9.0% | expand-content |
| `LOW_VOLUME` | 4,330 | 4.7% | monitor |
| `CTR_UNDERPERFORM` | 2,837 | 3.1% | fix-ctr |

**How an editor would use this tomorrow:** open the ranked queue, start from the top,
and treat each row's reason code as a starting hypothesis, not a verdict — verify the
specific issue (e.g. check the page actually ranks poorly, not just that the model
says so) before spending effort. Full intended-use, human-review, no-go list, and
monitoring/retrain triggers are documented in `work/notebooks/w07_action_playbook.ipynb`.

**Confidence and limits:** this is decision-support, not an automated action list.
Precision@50 = 0.96 (client-grouped) means the top of the queue is reliable; nothing
here should be auto-published, auto-deleted, or used client-facing without human
review. The label compares two 15-day halves of one month — a short baseline that
can't fully distinguish a real decline from normal search volatility.

## 8. Reproducibility

**Repo:** https://github.com/EmanHrustemovic/FlyRank-AI-Intership-ML-Track-

**Notebooks (run in order):**
1. `work/notebooks/w03_data_contract.ipynb` — data contract, grain/availability
   verification, leakage demo
2. `work/notebooks/w04_baseline_score.ipynb` — signal audit, baseline rule, ranked
   queue
3. `work/notebooks/w05_model.ipynb` — Logistic Regression vs Random Forest vs
   baseline, error analysis
4. `work/notebooks/w06_validation_audit.ipynb` — client-grouped re-evaluation,
   leakage audit, claim rewrite
5. `work/notebooks/w07_action_playbook.ipynb` — reason-code queue, intended use,
   human review, monitoring triggers

**To re-run from a fresh clone:** open each notebook in Google Colab, add a
Hugging Face read token as a Colab Secret (request dataset access at
`huggingface.co/datasets/FlyRank/internship-warehouse`), then Runtime → Run all.
Each notebook rebuilds its own data from the warehouse — no local data files are
required or committed.

**Random seed:** `random_state=42` used throughout (train/test splits, model fitting).

**Environment:** Google Colab default Python 3 runtime; key packages: `duckdb`,
`pandas`, `scikit-learn`, `numpy`. No `requirements.txt` pinning was needed beyond
Colab's default environment.

---

## Claims checklist before submitting

- [x] Language throughout uses observed / measured / directional / decision-support
      framing, not certainty claims.
- [x] Base rate (52.0%) reported alongside Precision@50 — the score is a lift over
      that base rate, not evaluated in isolation.
- [x] AUC / lift over baseline reported as the honest discrimination numbers
      (baseline AUC 0.544 vs. model AUC 0.649 on the honest split).
- [x] No causal claims — all findings are described as observed associations in this
      dataset, not causal mechanisms.
- [x] No claim of predicting or reverse-engineering Google's algorithm.
- [x] No client-identifying details anywhere (all IDs are salted hashes from the
      warehouse release itself).
- [x] Numbers in this report match a fresh re-run of the notebooks listed in Section 8.
