# Which Road Accidents End Badly? Predicting Severe Injury Accidents in Germany (2025)

**Team: The Random Forest Rangers**: Britt Schulze-Adesida · Jishnu Vasudevan · Micheal Eshak · Tiziano Cocciò · Zina Beek

When an injury accident happens on a German road, how likely is it that someone is **killed or seriously injured**? We use all police-recorded injury accidents from 2025 (Unfallatlas) together with official municipality data from Destatis to estimate this risk, using **only information that is known before an accident happens**.

## Business Case (Scenario C)

| | |
|---|---|
| **Stakeholders** | Municipal road planners, road-safety authorities, navigation and warning apps |
| **Question** | *If* an injury accident happens with these road users, at this place, time and conditions, how likely is it to be **severe**? |
| **Target** | `severe` = 1 if someone was killed or seriously injured (15.7 % of accidents), 0 if only slightly injured |
| **Metric** | **F1 score** for the severe class (with precision, recall and PR-AUC). Accuracy is misleading: always saying "not severe" is 84 % correct. |
| **Not used** | `accident_kind` and `accident_type`: they describe *how* the accident happened and are only known afterwards (leakage for prevention) |

## Data

| Source | Content |
|---|---|
| [Unfallatlas 2025](https://unfallatlas.statistikportal.de/) (Statistische Ämter des Bundes und der Länder) | Every accident with personal injury: place, time, light, road surface, road users, severity. Column guide: `data/DSB_Unfallatlas_EN.pdf` |
| [Destatis Gemeindeverzeichnis](https://www.destatis.de/DE/Themen/Laender-Regionen/Regionales/Gemeindeverzeichnis/_inhalt.html) (31.12.2025) | Population, area, density and official degree of urbanisation (urban / towns / rural) per municipality |
| [Destatis GENESIS 12411-0014](https://genesis.destatis.de/datenbank/online?sequenz=tabelleErgebnis&selectionname=12411-0014) | Population per state by age (used for the state-level EDA) |

**Shared team dataset:** `data/accidents_2025_with_population.csv`: **272,812 accidents × 41 columns**. 195 accidents (0.07 %) could not be matched to a municipality, mostly in uninhabited areas, and were left out.

## Key Findings

- **City vs countryside is the strongest factor:** 11.3 % of accidents are severe in cities, 16.2 % in towns and 24.3 % in rural areas, in every one of the 16 states. Most of the gap comes from the *same kind* of accident being more severe in rural areas, most likely because of higher speeds.
- **Who is involved matters next:** motorcycles 24.8 % severe (about 35 % on rural roads), trucks 21.0 %, pedestrians 19.2 %; accidents with a car are less often severe (13.7 %).
- **Time:** night (peak 26.3 % at 03:00) and weekends (Sunday 19.8 %) are riskier.

## Results (validation set)

| Approach | F1 | PR-AUC |
|---|---|---|
| Baseline: simple rule (motorcycle, pedestrian or no car) | 0.303 | n/a |
| Baseline: historical scenario lookup | 0.315 | 0.229 |
| Logistic regression (tuned) | 0.336 | 0.264 |
| Random forest (tuned) | 0.341 | 0.272 |
| **Gradient boosting (tuned, calibrated): final model** | **0.341** | **0.272** |
| Stacking ensemble | 0.344 | 0.273 |
| *Reference only: same model + accident kind/type (not usable)* | *0.375* | *0.300* |

- Every model beats both baselines; the final model finds **56 %** of severe accidents, and **1 in 4** flagged accidents is really severe (average: 1 in 6). In the 10 % highest-risk accidents, **1 in 3** is severe.
- All team notebooks reach the same level (**F1 ≈ 0.34**) with different code. Tuning (+0.002) and ensembles (+0.003) add very little: **the limit is the information in the data**, not the model.
- Population density is the most important feature (about 3× more than any other); the official urbanisation works almost as well, but adds nothing on top of density (`zb05.ipynb`).
- The probabilities are calibrated: mean predicted risk 15.7 % = observed 15.7 %.
- **Test set:** used only once, at the very end, with the final saved model (`models/random_forest_rangers_final_model.joblib`). Test score: *to be added after the final run.*

## Recommendations

1. **Prioritise rural roads**, especially for motorcycles: speed limits, safer curves and targeted campaigns.
2. **Use the model as a risk ranking**, not a yes/no decision; cities need their own threshold or model.
3. **Add the missing data:** road type and speed limits (e.g. OpenStreetMap) and several years of accidents.

**Data product idea:** a risk explorer for road planners: choose a place, time and road users and get the calibrated probability of a severe outcome, shown as a risk map, for targeted campaigns or live warnings in navigation apps.

**Limits:** the Unfallatlas does not record speed, road type, helmet or seatbelt use; density describes the whole municipality, not the road; one year only, random split.

## Repository Structure

| File | Owner | Content |
|---|---|---|
| **`Micheal_ML_Random_Forest_Rangers_Final.ipynb`** | Micheal | **Final team notebook:** business case, EDA, baselines, three models, tuning, ensembles, ablation, calibration, error analysis, post-accident reference, conclusion |
| `Micheal_Accident_Locations_2025_ML_Project.ipynb` | Micheal | First EDA and baseline model |
| `zb01.ipynb` | Zina | EDA and merge of Unfallatlas with Destatis municipality data → shared dataset |
| `zb02.ipynb` | Zina | Population by state and age (Destatis GENESIS) |
| `zb03.ipynb` | Zina | First Scenario C models and the effect of population density |
| `zb04.ipynb` | Zina | State-level EDA: population, age groups, urbanisation, accident kinds |
| `zb05.ipynb` | Zina | Urbanisation vs density as features, combining the team's best practices |
| `01_jishnu_EDA02.ipynb` | Jishnu | Detailed EDA |
| `Jishnu_eda_baseline.ipynb` | Jishnu | EDA and historical scenario baseline |
| `02_jishnu_modeling.ipynb`, `03_jishnu_modeling.ipynb` | Jishnu | Probability models, tuning, calibration, error analysis |
| `tiziano_eda.ipynb` | Tiziano | EDA on the raw data |
| `tiziano_splits.ipynb` | Tiziano | Train / validation / test split (70 / 15 / 15) → `data/splits/` |
| `tiziano_baselines.ipynb` | Tiziano | Linear baselines |
| `tiziano_ensembles.ipynb` | Tiziano | Bagging, boosting, voting and stacking |
| `tiziano_augmented.ipynb` | Tiziano | Educated guesses for speed, road type, helmet and seatbelt |
| `tiziano_pca.ipynb` | Tiziano | PCA before gradient boosting |
| `data/` | | Raw and processed data, splits |
| `models/` | | Saved final model |
| `results/` | | Saved tuning and ablation results |
| `01_assignment.md`, `02_kanban_board.md`, `03_eda-and-modeling.ipynb` | | Course template files |

## Setup and Reproduction

Requires Python 3.13 and [uv](https://docs.astral.sh/uv/).

```bash
git clone https://github.com/zinaF19/The-Random-Forest-Rangers.git
cd The-Random-Forest-Rangers
uv sync
code .
```

Open the notebooks from the project root and select the `.venv` kernel. To reproduce the final result, run `Micheal_ML_Random_Forest_Rangers_Final.ipynb` (it uses `data/accidents_2025_with_population.csv`, `data/city_accidents_2025.csv`, `data/population_states_licence_2025.csv` and `data/splits/`). Saved tuning results in `results/` let it run without repeating the slow searches.
