# March Madness Predictor

This repository builds a March Madness prediction pipeline from Kaggle competition data, creates processed team and game datasets, trains a machine learning model, and uses that model to simulate tournament brackets.

The workflow is notebook-driven and is intended to be run in a specific order:

1. `team_analysis.ipynb`
2. `game_analysis.ipynb`
3. `machine_learning_model.ipynb`
4. `matchup_simulation.ipynb`

## Repository Structure

- `data/march-machine-learning-mania-2025/`: raw Kaggle competition download for the 2025 dataset.
- `data/march-machine-learning-mania-2026/`: raw Kaggle competition download for the 2026 dataset.
- `data/relevant_data/`: the curated CSV files used directly by the notebooks.
- `data/processed/`: generated intermediate datasets written by the notebooks.
- `models/`: trained model artifacts.

## Data Source

The data comes from Kaggle's March Madness competition dataset, distributed through the March Machine Learning Mania competition pages. To reproduce or refresh the raw data:

1. Create a Kaggle account and generate an API token from your Kaggle account settings.
2. Install the Kaggle CLI if needed:

```bash
pip install kaggle
```

3. Download the competition files. For example:

```bash
kaggle competitions download -c march-machine-learning-mania-2026
```

4. Unzip the archive into a folder under `data/`, such as `data/march-machine-learning-mania-2026/`.
5. Copy the CSV files your notebooks need into `data/relevant_data/`.

If you prefer, you can also download the files directly from the Kaggle competition page in the browser and place them in the same directory structure used here.

## Files Used in `data/relevant_data/`

The curated `data/relevant_data/` directory contains the CSV files used by the notebooks in this project:

- `Cities.csv`
- `Conferences.csv`
- `MConferenceTourneyGames.csv`
- `MGameCities.csv`
- `MMasseyOrdinals.csv`
- `MNCAATourneyCompactResults.csv`
- `MNCAATourneyDetailedResults.csv`
- `MNCAATourneySeedRoundSlots.csv`
- `MNCAATourneySeeds.csv`
- `MNCAATourneySlots.csv`
- `MRegularSeasonCompactResults.csv`
- `MRegularSeasonDetailedResults.csv`
- `MTeamCoaches.csv`
- `MTeamConferences.csv`
- `MTeams.csv`

These are the working input files for the notebook pipeline. The notebooks read from `data/relevant_data/` and write derived outputs to `data/processed/`.

## Notebook Execution Order

Run the notebooks in the following order.

### 1. `team_analysis.ipynb`

This notebook loads team reference data from `data/relevant_data/`, including:

- `MTeams.csv`
- `MTeamConferences.csv`
- `MTeamCoaches.csv`
- `Cities.csv`

It joins those datasets into a combined team-level table and writes:

- `data/processed/team_data.csv`

### 2. `game_analysis.ipynb`

This notebook loads regular-season and NCAA tournament game results from `data/relevant_data/`, enriches them with team names, and performs exploratory analysis. It generates processed game-level datasets used later by the model notebook:

- `data/processed/RecentMRegularSeasonDetailedResults.csv`
- `data/processed/RecentMNCAATourneyDetailedResults.csv`

### 3. `machine_learning_model.ipynb`

This notebook uses the processed outputs from the earlier notebooks along with tournament seed data to engineer matchup features, train the prediction model, and save artifacts.

Inputs used here include:

- `data/processed/RecentMRegularSeasonDetailedResults.csv`
- `data/processed/RecentMNCAATourneyDetailedResults.csv`
- `data/relevant_data/MTeams.csv`
- `data/relevant_data/MNCAATourneySeeds.csv`

Outputs generated here include:

- `data/processed/MNCAATourneyMatchupFeatures.csv`
- `models/xgb_march_madness_model.joblib`

### 4. `matchup_simulation.ipynb`

This notebook loads the saved model and generated matchup features, then simulates bracket outcomes and upset scenarios.

It depends on artifacts created earlier in the pipeline, especially:

- `models/xgb_march_madness_model.joblib`
- `data/processed/MNCAATourneyMatchupFeatures.csv`
- `data/processed/team_data.csv`

It also uses tournament structure files from `data/relevant_data/`, including:

- `MNCAATourneySeeds.csv`
- `MNCAATourneySlots.csv`
- `MTeams.csv`

## Machine Learning Model

The project uses an XGBoost classifier (`XGBClassifier`) as the predictive model.

The model is trained on team-season feature differences built from regular-season box score aggregates. Features include team efficiency and possession-based metrics such as:

- effective field goal percentage (`eFG`)
- turnover rate (`TOVp`)
- offensive rebound rate (`ORBp`)
- free throw rate (`FTR`)
- offensive rating (`ORtg`)
- defensive rating (`DRtg`)
- win rate
- tournament seed difference

The trained model is saved to:

- `models/xgb_march_madness_model.joblib`

## End-to-End Outcome

If you run the notebooks in the required order, the repository will:

1. Read the curated Kaggle CSV files from `data/relevant_data/`.
2. Join and transform those source files into derived datasets in `data/processed/`.
3. Train and save an XGBoost March Madness prediction model in `models/`.
4. Use the saved model to simulate tournament brackets and upset scenarios.

## Suggested Python Packages

The notebooks use common data science libraries, including:

- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `scikit-learn`
- `xgboost`
- `joblib`

Installing those packages in a virtual environment before running the notebooks is recommended.

## Model Performance and Simulation Results

### Test-Set Performance

On a held-out test set built from recent regular-season and tournament games, the XGBoost model achieves approximately:

- Accuracy: 0.7836
- Log loss: 0.4796
- ROC AUC: 0.8461

These metrics are computed in `machine_learning_model.ipynb` using `accuracy_score`, `log_loss`, and `roc_auc_score` from scikit-learn.

### Bracket Simulation Summary

Using the trained model, `matchup_simulation.ipynb` simulates the full NCAA tournament bracket for the latest season and a championship game score distribution.

- Deterministic bracket champion: Arizona (Z01)
- Championship matchup: Florida (X01) vs Arizona (Z01)
- Simulated average final scores (10,000 simulations):
	- Florida: 73.6 points
	- Arizona: 76.5 points

### Comparing Metrics and Simulation Outcomes

- The test-set metrics indicate the model captures useful signal (ROC AUC ≈ 0.85) while maintaining reasonably well-calibrated probabilities (log loss ≈ 0.48).
- The bracket simulation uses these probabilities to propagate uncertainty through the tournament, producing a plausible champion (Arizona as a 1‑seed) and a close projected championship scoreline (roughly 76–74 in favor of Arizona).
- Together, the evaluation metrics and simulated bracket outcomes suggest the model is both statistically strong on historical data and qualitatively reasonable when translated into full-bracket predictions.

## After the Tournament

### Results

- Applied to the 2026 March Madness tournament, the model correctly predicted 22 of 32 games in the first round and 11 of 16 games in the second round. As the tournament progressed and matchups became more competitive, performance declined: the simulated bracket selected 2 of the 4 Final Four teams correctly and missed both teams in the championship game.
- In practice, this places the model in the range of a reasonable but not exceptional bracket. It is best used as an analytical tool for exploring scenarios and upset probabilities rather than as a fully automated bracket-picking system.