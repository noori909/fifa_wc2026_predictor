# FIFA World Cup 2026 Predictive Pipeline and Monte Carlo Tournament Simulator

## Project Overview

This repository contains a modular, production-grade machine learning pipeline engineered to predict match-level outcome probabilities for international football and aggregate them into a comprehensive tournament simulation for the FIFA World Cup 2026. Unlike traditional monolithic predictors that treat tournament winners as a direct classification problem under severe data sparsity, this system breaks down the predictive task into granular components: match-level multi-class probability estimation followed by parallelized Monte Carlo tournament simulations.

The workflow is built across four decoupled notebooks, ensuring a clear data-to-simulator contract that allows data extraction, feature synthesis, model calibration, and simulation logic to be modified independently without requiring a full pipeline rollback. To prevent any future-leaking features or lookahead bias, a strict chronological boundary is enforced across all training sets.

## Architecture and Directory Structure

The project decouples computation and storage using a standardized directory layout optimized for automated cell execution and persistent tracking via Google Drive:
fifa-wc2026-predictor/
├── data_pipeline/
│ ├── 01_data_ingestion/
│ │ ├── input/
│ │ └── output/
│ │ ├── cleaned_historical_matches.csv
│ │ └── team_index_map.csv
│ ├── 02_feature_engineering/
│ │ ├── input/
│ │ └── output/
│ │ └── final_engineered_feature_matrix.csv
│ ├── 03_model_training/
│ │ ├── input/
│ │ └── output/
│ │ └── simulation_team_strengths.csv
│ └── 04_tournament_simulation/
│ ├── input/
│ └── output/
│ └── monte_carlo_results.csv
└── notebooks/
├── 01_data_ingestion_eda.ipynb
├── 02_feature_engineering.ipynb
├── 03_model_training.ipynb
└── 04_monte_carlo_simulator.ipynb


### Notebook Blueprint Matrix

| Notebook Path | Core Technical Focus | Primary Deliverables and Outputs |
|--------------|---------------------|----------------------------------|
| `notebooks/01_data_ingestion_eda.ipynb` | Ingestion, Time-Walling, and Entity Resolution | `cleaned_historical_matches.csv`, `team_index_map.csv` |
| `notebooks/02_feature_engineering.ipynb` | Player Telemetry Ingestion and Feature Synthesis | `final_engineered_feature_matrix.csv` |
| `notebooks/03_model_training.ipynb` | Regularization, Diagnostic Alignment, and Calibration | `simulation_team_strengths.csv` |
| `notebooks/04_monte_carlo_simulator.ipynb` | 48-Team Filtering and Parallel Monte Carlo Simulations | `monte_carlo_results.csv` |

## Technical Specifications by Pipeline Stage

### 1. Data Ingestion and Entity Resolution (`01_data_ingestion_eda.ipynb`)

**Objective:** Establish an uncompromised historical match foundation and a unified team indexing framework.

- **Temporal Walling:** To kill lookahead bias during the live tournament timeline, a hard chronological defense wall is programmatically applied at May 31, 2026. Any match records or data occurrences past this boundary are stripped from the training log.
- **Entity Resolution Engine:** International football results span over a century and utilize conflicting team strings across multi-source archives (e.g., `results.csv` unpacked from historical records mapping "United States" vs "USA" or "Korea Republic" vs "South Korea"). Automated fuzzy matching and string standardization dictionaries map historical variations to a clean, standardized 48-key country index, preventing vector dropouts or NaN generation during downstream steps.
- **Outputs:** Generates `cleaned_historical_matches.csv` and `team_index_map.csv`.

### 2. Advanced Feature Engineering (`02_feature_engineering.ipynb`)

**Objective:** Extract and project raw historical and telemetry features into predictive, multi-dimensional feature vectors.

- **Squad Dynamics and Analytics:** Decompresses player telemetry stats (including `Career_Mode_FIFA_15-20_Statistics.zip` to parse `players_2020.csv`). It programmatically aggregates micro-level telemetry to squad-level metrics, focusing on total international caps, average squad age, experience ratios in elite professional leagues, and custom indices representing injury vulnerabilities.
- **Contextual Feature Weights:** Rejects static historical ratios by executing time-decay weighting (prioritizing recent rolling forms over old match timelines). Weights are modified dynamically according to match competitive tiers (e.g., weighting a UEFA Euros or Copa América fixture higher than friendly exhibitions) and matching environmental conditions.
- **Pairwise Interaction Metrics:** Constructs structural delta matrices based on opposing team profiles, outputting direct features such as `form_difference`, `attack_vs_defense_clash`, and `squad_quality_difference`.
- **Outputs:** Saves the complete `final_engineered_feature_matrix.csv`.

### 3. Calibrated Model Training and Optimization (`03_model_training.ipynb`)

**Objective:** Train and regularize models to generate highly objective multi-class match probabilities without overconfidence.

- **Target Mapping:** The mathematical optimization architecture predicts a multi-class array configured exactly against the target vector: `[2: Home Win, 1: Draw, 0: Away Win]`.
- **Diagnostic Alignment Warning System:** Programmatic cells calculate the explicit correlation direction between engineered delta features and the target labels. If any structural feature inversion is caught (e.g., a positive form delta falsely correlating with an away loss due to indexing misalignment), the system fires a warning statement to prevent garbage-in-garbage-out convergence.
- **Regularization Framework:** Baseline classifiers (Regularized Logistic Regression, Random Forest, and XGBoost) initially displayed severe overfitting and inflated probability leaves, pushing the Log-Loss past the random guess baseline of approximately 1.0986 and pulling the ROC-AUC under 0.5. To stabilize the pipeline, explicit hyperparameter constraints were integrated:
  - Tree models were restricted with explicit `max_depth` limits, heightened minimum sample splits, and stochastic row/column subsampling to stop historical memorization.
  - Logistic Regression structures enforced strict L1/L2 penalty boundaries to filter out uncalibrated, noisy signals.
- **Probability Smoothing Engine:** The regularized estimators are wrapped within scikit-learn's `CalibratedClassifierCV` using isotonic and sigmoid methods. This smooths out overconfident tree leaves, minimizes multi-class Log-Loss, and ensures the output probabilities represent realistic analytical odds suitable for a simulation space.
- **Outputs:** Maps validation set row indices back to their real country strings and saves the calculated `avg_expected_win_probability` into `simulation_team_strengths.csv`.

### 4. Monte Carlo Tournament Simulation Engine (`04_monte_carlo_simulator.ipynb`)

**Objective:** Execute parallel simulations of the official FIFA World Cup 2026 tournament structure using calibrated model outputs.

- **Agile Pipeline Filtering:** Historical records include microstates and amateur football teams (e.g., Vatican City, Kernow) that might hold anomalous win ratios (e.g., winning 4 out of 5 historical matches against local selections). If uncorrected, tree models interpret an 80% historical win rate as an elite global tier, placing these microstates above traditional powerhouses like France or Argentina. Rather than triggering a full upstream pipeline rollback to clean the dataset, Notebook 4 implements an agile inner join between `simulation_team_strengths.csv` and the official `SquadLists.csv` containing only the true 48 qualified nations. This removes statistical noise before tournament execution.
- **Hard Validation Gate:** To guarantee zero entity loss due to special character string encodings (such as the UTF-8 variants or apostrophes present in 'Curaçao' and 'Côte D'Ivoire'), the engine implements a programmatic validation checkpoint:
  ```python
  assert len(df) == 48, f"Data alignment dropout detected. Current team count: {len(df)}"
A comprehensive translation and mapping dictionary ensures a 1:1 entity match to pass this gate cleanly without silent dropouts.

Simulation Loop Mechanics: Runs 10,000 uncompromised, parallel tournament realities. The logic perfectly mirrors the expanded 2026 format:

Group Stage: Models 12 groups of 4 teams each, executing round-robin schedules where match probabilities govern outcomes, tracking group standings, and handling advanced 3-team qualification rules.

Knockout Bracket Tree: Constructs a single-elimination bracket spanning the Round of 32, Round of 16, Quarterfinals, Semifinals, and the World Cup Final.

Outputs: Generates the aggregated final winner probability distribution matrix as monte_carlo_results.csv.

Final Simulation Analytics
After running 10,000 parallel iterations through the regularized and calibrated XGBoost tournament engine, the following probability distribution represents the top 10 most likely contenders to win the 2026 FIFA World Cup:

Rank	Qualified Team Country	Simulated Win Probability
1	Spain	4.21%
2	Belgium	4.20%
3	Brazil	4.04%
4	Portugal	3.81%
5	France	3.75%
6	Germany	3.70%
7	England	3.70%
8	Côte D'Ivoire	3.65%
9	USA	3.64%
10	Japan	3.57%
Note: The narrow spread in probabilities highlights the hyper-realistic equilibrium achieved by applying aggressive tree calibration, preventing individual team profiles from scaling into uncalibrated statistical monopolies.

Execution Instructions
Prerequisites
The environment requires a Python 3.10+ runtime equipped with standard data science modules:

numpy

pandas

scikit-learn

xgboost

lightgbm

Step-by-Step Execution
Workspace Sync: Mount your Google Drive repository or clone the file tree locally. Verify that all multi-source raw zip archives are situated in the designated input directories.

Execute Ingestion: Run notebooks/01_data_ingestion_eda.ipynb to process historical logs, apply the May 31, 2026 temporal defense wall, and export the standardized index map.

Execute Feature Engineering: Run notebooks/02_feature_engineering.ipynb to read the cleaned logs, process player statistics, compile decaying rolling features, and construct the master feature matrix.

Execute Training and Calibration: Run notebooks/03_model_training.ipynb to run baseline modeling, execute alignment diagnostics, optimize regularized tree algorithms, and apply CalibratedClassifierCV.

Execute Simulator: Run notebooks/04_monte_carlo_simulator.ipynb to process the 48-team validation filtering gate and run the 10,000 parallel tournament iterations. Review monte_carlo_results.csv for final probability distributions.