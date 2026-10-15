# UrbanHeat AI (HeatSense)

**Study Area:** Ahmedabad, Gujarat, India
**Project:** An AI-powered urban heat island detection, explainability and mitigation decision-support system for Ahmedabad.

UrbanHeat AI uses satellite-derived urban indicators and machine learning to identify heat-stress hotspots, explain heat drivers, estimate population-aware vulnerability, and compare cooling-intervention scenarios for priority zones across Ahmedabad.

## Live Dashboard & Local Execution

The interactive Streamlit dashboard provides views of urban heat hotspots, environmental drivers, heat vulnerability, cooling-intervention simulations, model validation, and priority zones.

To run the application locally:

```bash
cd C:\Study\SEM_5\ML\Project\urbanheatplan

# Option 1: with uv
uv run streamlit run app.py

# Option 2: with plain Python (use the project virtualenv)
python -m streamlit run app.py
```

Dependencies are pinned in `requirements.txt` (Python 3.11 virtualenv). Install with:

```bash
pip install -r requirements.txt
```

To reproduce the full analysis, run the notebooks in order (`notebooks/01` → `notebooks/13`); Notebook 13 regenerates the CSVs in `outputs/dashboard_data/` consumed by the dashboard.

## Study Area

Ahmedabad, Gujarat, India (approx. latitude 22.90–23.15, longitude 72.46–72.70; map centre 23.0225, 72.5714). All observations, training data, predictions and map outputs in this repository correspond to Ahmedabad. Geographic validation confirms sampled coordinates fall inside the Ahmedabad extent.

## Dataset

- Source: Google Earth Engine export, `data/raw/ahmedabad_uhi_master.csv` — **98,078 sampled locations**.
- Raw schema: `system:index, Elevation, LST, LandCover, Latitude, Longitude, NDBI, NDVI, Population, .geo`.
- Intended source products: Landsat 8/9 Collection 2 Level 2 (land surface temperature), Sentinel-2 Surface Reflectance Harmonized (NDVI, NDBI), SRTM (elevation), WorldPop (population estimates), ESA WorldCover (land-cover classification). Verify collection IDs, dates, bands, scaling factors and resolutions against the export metadata before reuse.
- LST is stored in **°C** with satellite scaling factors already applied. `Population` is a gridded exposure proxy, not a census headcount — see Limitations. `LandCover` holds numeric codes mapped to named classes and one-hot encoded during feature engineering (see `notebooks/04_feature_engineering.ipynb`). `.geo` carries redundant point geometry; coordinates are retained for mapping but excluded from model inputs.

## Methodology

### Feature engineering

Model inputs (11 features, fixed order in `outputs/models/xgboost_v3_feature_order.csv`):

- `NDVI`, `NDBI`, `Elevation`, `Population`
- One-hot land cover: `LandCover_Bare_sparse_vegetation`, `LandCover_Built-up land`, `LandCover_Cropland`, `LandCover_Grassland`, `LandCover_Permanent_water_bodies`, `LandCover_Shrubland`, `LandCover_Tree cover`

Target variable: `LST` (°C). `Latitude`/`Longitude` are excluded from model predictors and used only for spatial grouping, mapping and visualisation.

### Model and validation

- Models compared: Linear Regression (baseline), Random Forest Regressor, XGBoost Regressor. Primary model: **XGBoost Regressor** (`outputs/models/xgboost_v3_landcover_model.pkl`).
- Validation: **spatial block cross-validation** — complete spatial blocks are held out for testing to reduce leakage between nearby training and test samples.
- Ahmedabad performance (from `outputs/dashboard_data/dashboard_kpis_v3.csv`): **R² ~0.57, MAE ~1.67 °C**, over 98,078 sampled locations.
- SHAP analysis (`outputs/reports/shap_feature_importance_v3.csv`) explains each feature's contribution to predicted LST; built-up intensity (NDBI) is the dominant heat driver.
- The export contains no per-observation dates, so the system performs **spatial LST estimation and scenario analysis**, not validated future temporal forecasting.

### Hotspots, vulnerability and scenarios

- Hotspot categories (Low / Moderate / High / Very High) are derived from documented observed/predicted LST rules; the dashboard tracks 10 high-priority hotspot zones.
- The Heat Vulnerability Index combines predicted heat hazard with population exposure — the hottest locations are not necessarily the most vulnerable.
- Cooling scenarios simulated with the trained model: **+20% green cover (urban greening)** and **cool/permeable surfaces**. Reported temperature changes are model-based planning estimates for comparing interventions, not measured outcomes.

## Maps

All interactive maps use **OpenStreetMap-based tiles** (`open-street-map` style, no API keys, with OSM attribution). Point layers support scatter and LST-gradient heatmap views; the Streamlit AI simulator runs live XGBoost inference on user-adjusted greening, tree plantation, cool-roof, permeable-surface and built-up-reduction settings.

## Repository structure

```text
.
├── app.py                        # Streamlit dashboard
├── src/                          # cleaning, features, model, hotspot, eda, utils
├── notebooks/                    # 01 audit → 13 dashboard data preparation
├── data/raw/                     # ahmedabad_uhi_master.csv (GEE export)
├── data/processed/               # cleaned, featured, model-ready, scenario outputs
├── outputs/dashboard_data/       # CSVs consumed by app.py
├── outputs/models/               # trained XGBoost model + feature order
├── outputs/reports/              # priority, vulnerability, SHAP, audit reports
├── outputs/plots/                # SHAP and evaluation visualisations
├── outputs/maps/                 # interactive HTML maps
├── requirements.txt
└── README.md
```

## Limitations

- Satellite LST is not air temperature and does not equal human-felt heat.
- Population is an exposure proxy, not a full social-vulnerability measure.
- Cooling values are model-based scenario estimates, not verified real-world outcomes; field validation, engineering review and municipal feasibility checks are required before implementation.
- Single undated export: hotspot mapping and scenario comparison are supported; temporal forecasting claims are not.

## Attribution

Adapted from the urban heat-stress hotspot-detection workflow originally developed for Chhatrapati Sambhajinagar (see `https://github.com/priyaldeshmukh2005/urban-heat-stress-hotspot-detection`). Inherited modelling, explainability and dashboard code is reused from that project; all data, trained models, metrics and geographic outputs here are regenerated for Ahmedabad. Preserve the original repository's licence terms when redistributing inherited code.
