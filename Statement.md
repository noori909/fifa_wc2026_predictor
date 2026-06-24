## Assignment Core & Objectives

- **Primary Goal:** Predict FIFA World Cup winner using any 3 ML models.
- **Data Scope:** Rich, multi-source data including teams, players, playing styles, player stats, win rates, formations, ages, preparations, coaches, expert analyses, environmental/venue factors, injuries, substitutes, and availability.
- **Preferred Professional Approach** (not direct tournament winner classification due to sparse data):
  - Predict match-level outcome probabilities (win/draw/advance) first.
  - Aggregate upward to contender rankings and overall winner probabilities.
- **Feature Engineering Focus:** Squad strength aggregates, injury impact indices, rolling recent form with weights, pairwise differences/interactions, coach features, style metrics, etc.
- **Bias Prevention:** Avoid lookahead bias by using only pre-match data and enforcing temporal alignment when merging datasets.
- **Validation Strategy:** Time-based / walk-forward validation preferred, while incorporating ~80/10/10 train/validation/test proportions respecting chronology.
- **Context:** World Cup 2026 is currently ongoing (mid-June 2026 context) — models trained on historical data.

## Model Selection Discussion

**Suggested Trio:**
1. **Regularized Logistic Regression** — Interpretable baseline.
2. **Random Forest** — Nonlinear ensemble.
3. **XGBoost** — Strong tabular performer.

**Evaluation Metrics:** Log loss, Brier score, accuracy, calibration, feature importance, etc.

**Comparison Section:** Notebook includes a dedicated comparison highlighting strengths/weaknesses, especially regarding probability quality and aggregation performance.

## Key Risks & Mitigations Discussed

| Risk | Mitigation |
|------|------------|
| Lookahead bias | Rolling features, temporal splits |
| Multicollinearity | Feature selection, regularization |
| Missing/inconsistent data across sources | Careful merging, imputation strategies |
| Era differences | Time-decay weighting, temporal alignment |
| Overfitting | Regularization, calibration, walk-forward validation |
| Sparse tournament-level labels | Match-level predictions with upward aggregation |

## Project Workflow & Notebook Structure

**Sequential Pipeline:**
1. Data loading/integration
2. Exploratory Data Analysis (EDA)
3. Feature Engineering
4. Data Preparation & Splitting
5. Model Training & Evaluation
6. Model Comparison
7. Match-Level Aggregation
8. Tournament Simulation & Results

**Delivery Format:** Detailed, professional Colab notebook with thorough markdown sections and code comments throughout.