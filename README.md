# 🔥 LIVE FIRE INTELLIGENCE — Wildfire Risk Monitor

An end-to-end wildfire ignition-risk monitoring system that combines **Google Earth Engine (GEE)** environmental data with an **XGBoost machine-learning model** and a web-based dashboard.

The system accepts a geographic location, target date, and prediction-grid size, retrieves the most recent usable Earth Engine observations, extracts environmental features, and returns a spatial wildfire ignition-risk probability grid.

> **Status:** 🟢 Live GEE data + XGBoost prediction  
> **Backend:** Python / FastAPI / Uvicorn  
> **ML model:** XGBoost  
> **Frontend:** HTML / CSS / JavaScript  
> **Spatial resolution:** approximately 2 km per cell

---

## ✨ Features

- 🌍 Live environmental data through Google Earth Engine
- 🔥 Spatial wildfire ignition-risk prediction
- 🤖 XGBoost probability model
- 🗺️ Grid-based risk visualization
- 📍 User-selectable latitude and longitude
- 📅 User-selectable target date
- 🔢 Configurable prediction grid: `16 × 16`, `32 × 32`, or `64 × 64`
- 📊 Peak and average predicted probability
- 🛰️ Data-window and Earth Engine freshness information
- ⚡ FastAPI REST API with interactive Swagger documentation
- 💻 Browser-based wildfire risk dashboard
- 🧪 Automated tests with pytest

---

## 🏗️ System Architecture

```text
                         ┌──────────────────────┐
                         │      Web Browser      │
                         │ HTML / CSS / JS UI    │
                         └──────────┬───────────┘
                                    │
                                    │ POST /predict
                                    ▼
                         ┌──────────────────────┐
                         │      FastAPI         │
                         │ wildfire.service     │
                         └──────────┬───────────┘
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                     ▼                             ▼
          ┌─────────────────────┐       ┌─────────────────────┐
          │ Google Earth Engine │       │   XGBoost Model     │
          │ Environmental Data  │──────▶│ Ignition Probability│
          └─────────────────────┘       └──────────┬──────────┘
                                                    │
                                                    ▼
                                      ┌─────────────────────────┐
                                      │ Spatial Risk Probability│
                                      │       Grid / JSON       │
                                      └────────────┬────────────┘
                                                   │
                                                   ▼
                                      ┌─────────────────────────┐
                                      │     Web Dashboard       │
                                      │ Risk Map + Statistics   │
                                      └─────────────────────────┘
```

---

## 📁 Project Structure

```text
wildfire-main/
│
├── wildfire/
│   ├── service.py              # FastAPI prediction service
│   ├── dashboard.py            # Dashboard / batch prediction logic
│   └── ...                     # Data extraction and ML modules
│
├── wildfire_models/
│   ├── west_coast_wildfire_model.json
│   └── feature_meta.json
│
├── wildfire_data/              # Local/generated data where applicable
├── wildfire_output/            # Generated prediction outputs
├── tests/                      # Automated tests
│
├── web/                        # Frontend files, if present
│   ├── index.html
│   ├── app.js
│   └── style.css
│
├── docs/                       # Project documentation
├── pyproject.toml              # Python project configuration
├── README.md
└── .gitignore
```

---

## 🧰 Requirements

- Python 3.10+
- Google Earth Engine account
- A registered Google Cloud / Earth Engine project
- Earth Engine Python API
- XGBoost
- FastAPI
- Uvicorn
- NumPy / Pandas
- Pytest

A virtual environment is recommended.

---

## 🚀 Installation

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd wildfire-main
```

### 2. Create a virtual environment

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 3. Install the project

If the repository contains the project configuration:

```powershell
python -m pip install -e .
```

If required by the project:

```powershell
python -m pip install numpy pandas xgboost fastapi uvicorn earthengine-api pytest
```

---

# 🌍 Google Earth Engine Setup

The prediction service uses live Earth Engine observations rather than silently replacing missing Earth Engine data with synthetic values.

### 1. Authenticate Earth Engine

```powershell
earthengine authenticate
```

Follow the browser authorization process.

A successful authentication looks like:

```text
Google Earth Engine authorization successful!
```

### 2. Register/use an Earth Engine project

Set the Earth Engine project from PowerShell:

```powershell
earthengine set_project YOUR_PROJECT_ID
```

For example:

```powershell
earthengine set_project west-coast-wildfire
```

You can also provide the project to the application through the environment:

```powershell
$env:GEE_PROJECT = "YOUR_PROJECT_ID"
```

> Do **not** commit private credentials, OAuth tokens, service-account keys, or `.env` files containing secrets.

---

# 🧠 Machine Learning

The project uses **XGBoost** to estimate wildfire ignition probability for individual spatial cells.

The model uses environmental and geographic predictors obtained from Earth Engine. Depending on the configured extraction pipeline, these include variables such as:

- Temperature
- Dew point
- Soil moisture
- Vegetation / NDVI-related information
- Topographic variables
- Slope
- Drought / dryness indicators
- Other Earth Engine-derived environmental features

Fire observations are used as the target/label within the project's training pipeline.

The trained model is stored as:

```text
wildfire_models/west_coast_wildfire_model.json
```

Feature metadata is stored as:

```text
wildfire_models/feature_meta.json
```

---

# 🏋️ Training

The project includes a training module.

To see the available options:

```powershell
python -m wildfire.train --help
```

For a synthetic-data training run used for development/testing:

```powershell
python -m wildfire.train --synthetic
```

A successful training run saves the model and feature metadata under:

```text
wildfire_models/
```

---

# ⚡ Run the API

Start the FastAPI service:

```powershell
python -m wildfire.service --port 8000
```

The API will be available at:

```text
http://127.0.0.1:8000
```

Interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

OpenAPI specification:

```text
http://127.0.0.1:8000/openapi.json
```

Health check:

```text
http://127.0.0.1:8000/health
```

A healthy response should indicate that the model and feature metadata are present.

---

# 🔮 Prediction API

The main endpoint is:

```text
POST /predict
```

Example request:

```json
{
  "lat": 37.7749,
  "lon": -122.4194,
  "target_date": "2026-10-02",
  "grid": 32
}
```

Example response structure:

```json
{
  "lat": 37.7749,
  "lon": -122.4194,
  "grid": 32,
  "pixel_km": 2,
  "bounds": {
    "west": -122.7841,
    "south": 37.4866,
    "east": -122.0547,
    "north": 38.0632
  },
  "window": {
    "start": "2026-09-25",
    "end": "2026-10-02",
    "days_behind_target": 0
  },
  "data_source": "live_gee",
  "fire_probability": [
    ["..."],
    ["..."]
  ]
}
```

The `fire_probability` field contains the predicted ignition probability for each cell in the spatial grid.

---

# 🗺️ Web Dashboard

The frontend provides a browser-based interface for:

- Latitude
- Longitude
- Target date
- Grid size
- Wildfire-risk analysis
- Peak probability
- Average probability
- Data window
- Spatial resolution
- Prediction details
- Risk visualization

The dashboard is designed around a **LIVE FIRE INTELLIGENCE** interface and can display the model's spatial probability grid over a geographic map.

If the frontend is served separately from FastAPI, run it from the directory containing `index.html`.

For example:

```powershell
python -m http.server 5500
```

Then open:

```text
http://127.0.0.1:5500
```

---

# 📊 Risk Interpretation

The model returns a probability for each spatial cell.

The dashboard can group the continuous probabilities into visual risk categories:

```text
Low       → lower predicted probability
Medium    → moderate predicted probability
High      → elevated predicted probability
Extreme   → highest predicted probability
```

These visual categories are intended for communication and visualization. They should not be interpreted as guaranteed wildfire occurrence.

---

# 🧪 Testing

Run the test suite with:

```powershell
pytest tests/ -q
```

Some tests may depend on the exact trained model artifact and expected validation values.

For API-level testing, the Swagger UI is available at:

```text
http://127.0.0.1:8000/docs
```

---

# 🔐 Security and Credentials

Never upload these to GitHub:

```text
*.json
```

unless they are known to contain only non-secret model metadata/artifacts.

In particular, never commit:

```text
.env
credentials.json
service-account-key.json
OAuth tokens
private API keys
```

Check `.gitignore` before pushing the repository.

If credentials were accidentally committed, revoke/rotate them immediately.

---

# 📚 Data Sources

The model relies on multi-source environmental and geospatial information accessed through **Google Earth Engine**.

The project can incorporate Earth-observation and environmental variables such as:

- NASA/USGS satellite-derived products
- ERA5-Land meteorological variables
- Soil-moisture observations
- Digital elevation / slope information
- Vegetation indices
- Fire-detection observations

Exact datasets and preprocessing should be documented according to the final configuration used for the submitted experiment.

---

# 📖 Related Research

The project is related to machine-learning approaches for wildfire detection, ignition-risk prediction, and fire-spread prediction.

If the repository uses the **Next Day Wildfire Spread (NDWS)** dataset/component, cite:

> Huot, F., Hu, R. L., Ihme, M., Burge, J., Ellicott, E., Kumar, A., Lattimer, B., McGovern, A., Nogueira, M., Prata, A. T., et al. “Next Day Wildfire Spread: A Machine Learning Dataset to Predict Wildfire Spreading from Remote-Sensing Data.” IEEE International Conference on Data Mining (ICDM), 2022.

Use the exact citation required by your project's final implementation and report.

---

# 🖥️ Example Workflow

```text
1. User enters latitude, longitude and target date
                    ↓
2. Frontend sends POST /predict
                    ↓
3. FastAPI validates the request
                    ↓
4. Earth Engine retrieves environmental observations
                    ↓
5. Features are prepared for each spatial cell
                    ↓
6. XGBoost predicts ignition probability
                    ↓
7. API returns the probability grid
                    ↓
8. Dashboard visualizes the risk
```

---

# ⚠️ Limitations

This project is a **research/prototype wildfire-risk monitoring system**.

Predictions should not be treated as:

- guaranteed wildfire occurrence
- official emergency warnings
- evacuation instructions
- a replacement for local fire authorities
- a substitute for professional fire-weather analysis

Model performance depends on:

- data availability
- Earth Engine observation latency
- feature quality
- spatial resolution
- training-label quality
- geographic coverage
- model calibration

---

# 👩‍💻 Development

Useful commands:

```powershell
# Activate environment
.\.venv\Scripts\Activate.ps1

# Check API command
python -m wildfire.service --help

# Start API
python -m wildfire.service --port 8000

# Train model
python -m wildfire.train --help

# Run tests
pytest tests/ -q
```

---

# 📜 License

See the repository's `LICENSE` file for the applicable license.

---

## ⭐ Project Summary

**LIVE FIRE INTELLIGENCE** is an end-to-end wildfire ignition-risk prediction system that connects **Google Earth Engine**, **XGBoost**, **FastAPI**, and a browser-based dashboard to transform environmental observations into an interactive spatial wildfire-risk map.

Built for research, demonstration, and machine-learning experimentation.
