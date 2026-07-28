# Growth / Recovery / Momentum Prediction — Execution Plan
**Lane:** Freestyle — FlyRank ML Internship
**Owner:** Michael Adesiyan
**Research Question:** Can we predict which pages are likely to decline, recover, or gain momentum?

---

## 1. Core Label Shape

```
prior feature window -> future target window
```

Chosen framing options (pick one, document the choice in `w01`):
- Prior 90 days of features -> next 30 days decline
- Prior 28 days of features -> next 28 days growth
- Prior state after a refresh -> later recovery signal (only if data safely supports it)

**Golden rule:** the feature window must never overlap the target window. This lane "lives or dies on clean future-window labels and strict leakage control."

---

## 2. Data Sources

| Table | Rows | Grain | Use |
|---|---|---|---|
| `dim_clients` | 104 | one row per client | check `gsc_data_start` / `ga4_data_start` before defining windows |
| `dim_content` | 519,606 | one row per content item | joins, metadata |
| `fact_content_daily_performance` | 78,835,655 | daily x client x content | build feature + target windows here |
| `fact_content_query_90d` | 2,414,248 | client x content x query hash | optional query-mix features |

- Dataset: `FlyRank/internship-warehouse` (Hugging Face, gated, instant approval)
- Date range: 2025-01-27 to 2026-06-30
- Only 9 of 70 clients with daily data have 12+ months history (needed for seasonality checks)
- **Dev tip:** iterate on `fact_content_daily_performance_sample` (~11.7M rows, latest full month) to avoid HF rate limits (HTTP 429). Only hit the full 78.8M table for the final pass.

---

## 3. Notebook-by-Notebook Plan

### `w01_research_question.ipynb`
- Write the final research question and pick the exact label shape (window lengths).
- Draft the data contract: which tables, which fields, which exclusions.
- State the decision this project supports (e.g. "rank pages for review before they decline further").

### `w02_signal_audit.ipynb`
- Load `dim_content` + `fact_content_daily_performance` (sample table first).
- Classify every column: join key / observed signal / derived measurement / target-proxy / excluded (never use `health_score`, `priority_score`, `action_type`).
- Check per-client `gsc_data_start` / `ga4_data_start` before trusting any window.

### `w03_data_prep_windows.ipynb`
- Use the DuckDB workflow (per repo guidance) to build feature window (e.g. days 0-89) and target window (e.g. days 90-119) per content item, per client.
- Build the label: decline / growth / recovery flag from future observed outcomes only.
- Run the full leakage checklist:
  - Any features calculated after the decision point?
  - Feature window overlapping target window?
  - Any product flag slipped in as a feature?
  - Any derived field secretly encoding the target?
  - Related rows split across train/test in a way that makes it too easy?

### `w04_baseline.ipynb`
- Build a transparent, explainable baseline (simple threshold rule on trend/volume — no ML).
- Record baseline precision@K, recall, average precision as the number to beat.

### `w05_model.ipynb`
- Try logistic regression -> decision tree -> random forest / gradient boosting, in that order.
- Use client-grouped and/or time-aware validation (train on past, test on future; keep whole clients out of training).
- Calibrate and review the probability threshold.

### `w06_validation.ipynb`
- Precision@20 / Precision@50 depending on realistic review capacity.
- Manually inspect top 20 predictions — do they make sense?
- Re-run the leakage checklist against the final model.
- Confirm the model beats the baseline; if not, document what was learned.

### `w07_action_playbook.ipynb`
- Turn model output into a ranked future-risk / future-opportunity list.
- Attach reason codes per prediction.
- Write the plain-words explanation of what the model can and cannot claim (no causal claims, no "refresh caused recovery" without a real experiment).

### `capstone.ipynb`
- Assemble the full story: problem, grain, data, features vs labels vs excluded fields, baseline, model, validation, top recommendations, reason codes, safe vs unproven claims.
- Public-safe check: no client names, domains, URLs, raw queries, or causal claims.
- Deploy as a public research page; add its URL to `submission/paper_url.txt`.

---

## 4. Do-Not-Do List (Lane-Specific)

- Do not use target-window metrics as features.
- Do not let pages from the same client land in both train and test without a grouped check.
- Do not call seasonal movement "model skill."
- Do not rebuild `health_score` / `priority_score` / `action_type` and feed it back in as a feature.
- Do not claim a refresh caused a recovery without a real experiment.

---

## 5. Capstone Self-Check (Final Gate)

- [ ] Problem and decision/ranking clearly stated
- [ ] Row grain defined
- [ ] Data tables and fields documented (features / labels / context / excluded / leakage risks)
- [ ] Baseline built and scored
- [ ] Model chosen and justified
- [ ] Validation design matches the problem (time-aware / client holdout)
- [ ] Metric matches the real decision (precision@K, recall, etc.)
- [ ] Model vs baseline comparison documented
- [ ] Top recommendations + reason codes produced
- [ ] Safe vs unproven claims separated
- [ ] Repo is fully rerunnable end to end
