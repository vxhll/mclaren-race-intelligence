# McLaren Race Intelligence

An AI-powered Formula 1 race intelligence platform built with Google Agent Development Kit (ADK), Gemini, BigQuery, BigQuery ML, and Python.

The project combines historical Formula 1 data preparation, analytical views, an ML podium prediction workflow, and a multi-agent assistant for race-data questions and visualizations.

## Achievement

**Google Cloud Labs: OnAir — Lab Completed**

Completed the *Build an AI-Powered F1 Analytics Agent* lab.

![Google Cloud Labs: OnAir — Build an AI-Powered F1 Analytics Agent lab completion badge](certifications/google-cloud-labs-onair-f1-analytics-agent.png)

## Architecture

```text
User
 |
 v
Root ADK Agent (Gemini)
 |-- BigQuery Toolset
 |     `-- Query Formula 1 analytical views
 |
 |-- Podium Prediction Tool
 |     `-- BigQuery ML model (ML.PREDICT)
 |
 `-- Visualization Agent
       `-- Python code execution and chart generation

Data pipeline
Kaggle F1 dataset / Jolpica API
       |
       v
Pandas data preparation and updates
       |
       v
Google Cloud Storage (Parquet + profiles)
       |
       v
BigQuery tables and analytical views
       |
       v
BigQuery ML podium prediction model
```

## Key components

- **Google ADK multi-agent system:** A root race-intelligence agent coordinates BigQuery access, podium predictions, and a visualization sub-agent.
- **Gemini:** Supports natural-language analysis and visualization planning.
- **BigQuery:** Stores and queries historical Formula 1 data through analytical views.
- **BigQuery ML:** Trains a logistic-regression classifier to predict whether a McLaren driver will finish on the podium.
- **Python and Pandas:** Prepare, clean, update, and analyze race data.
- **Google Cloud Storage:** Stages Parquet data and McLaren driver/team profiles.
- **Data refresh workflow:** Uses the Jolpica API to identify and fetch newer race rounds and refresh derived data and profiles.

## Machine learning workflow

The SQL model in `sql/create_model.sql` uses pre-race features, including:

- Grid position
- Driver championship position entering the race
- Constructor championship position entering the race
- Driver's historical average finish at the circuit
- McLaren's historical podium rate at the circuit
- Driver's rolling average finish over the previous five races

The model is trained on data through the 2023 season. The agent's default prediction workflow targets the 2024 season, with later seasons also usable when the corresponding data is available. The notebook and SQL files should be run in the intended order and against a correctly provisioned BigQuery environment.

## Repository structure

```text
.
├── agent/
│   ├── mclaren_agent/
│   │   ├── __init__.py
│   │   ├── agent.py
│   │   └── prompts.py
│   ├── requirements.txt
│   └── setup.sh
├── notebooks/
│   ├── mclaren_data_prep.ipynb
│   └── mclaren_data_update.ipynb
├── sql/
│   ├── create_model.sql
│   ├── evaluate_model.sql
│   ├── create_views.sh
│   └── views.sql
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

- A Google Cloud project with billing enabled
- BigQuery and Vertex AI access
- A Google Cloud Storage bucket that you own and can write to
- Python 3.10+ (use a version compatible with the installed Google ADK dependencies)
- Google Cloud CLI and Application Default Credentials
- Kaggle credentials if running the initial data preparation notebook

You must create/configure your own cloud resources. The notebooks originally target a managed Colab Enterprise environment and expect suitable IAM permissions. Replace the `YOUR_GCS_BUCKET` placeholder in the notebooks with your own bucket name before running them.

## Setup: ADK agent

From the repository root:

```bash
# Configure your Google Cloud project
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login

# Set environment variables (Linux/macOS)
export PROJECT_ID="YOUR_PROJECT_ID"
export REGION="us-central1"
export GOOGLE_GENAI_USE_VERTEXAI="1"
export GOOGLE_CLOUD_LOCATION="global"

# Install agent dependencies
cd agent
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Start the ADK web interface
adk web
```

On Windows PowerShell, activate the environment with:

```powershell
.\.venv\Scripts\Activate.ps1
$env:PROJECT_ID = "YOUR_PROJECT_ID"
$env:REGION = "us-central1"
$env:GOOGLE_GENAI_USE_VERTEXAI = "1"
$env:GOOGLE_CLOUD_LOCATION = "global"
```

The agent also needs the BigQuery dataset, tables, views, and model referenced by its prompts and prediction query. Those resources are not created automatically by starting the ADK web server.

## Data and SQL workflow

1. Run `notebooks/mclaren_data_prep.ipynb` in a suitably configured Colab Enterprise environment to prepare the initial data and profile assets.
2. Load the staged Parquet files into your BigQuery dataset.
3. Run `sql/create_views.sh` after setting `PROJECT_ID` to create the analytical views from `sql/views.sql`.
4. Run `sql/create_model.sql` to train the podium prediction model.
5. Run the queries in `sql/evaluate_model.sql` to inspect model evaluation metrics, weights, and the confusion matrix.
6. Use `notebooks/mclaren_data_update.ipynb` to refresh data when new race results become available.

Follow the notebook instructions and verify the schema and IAM permissions in your own environment before running cloud operations.

## Security

- Never commit `.env` files, Kaggle API keys, service-account keys, access tokens, or other credentials.
- Use environment variables, Colab Secrets, or Google Cloud Secret Manager for credentials.
- Use your own Google Cloud project and storage bucket; do not rely on classroom resources.
- Notebook outputs have been cleared in this repository copy to avoid publishing runtime information.

## Data sources and acknowledgements

The notebooks reference the Ergast-derived Formula 1 World Championship dataset distributed through Kaggle and the Jolpica Ergast-compatible API for subsequent race updates. Check the source dataset/API terms and attribution requirements before redistribution or commercial use.

## Notes

This repository contains the project code, notebooks, and SQL assets. It does not include the underlying race dataset, cloud-hosted BigQuery tables, trained model artifacts, or deployed cloud services. Actual runtime results depend on the cloud resources and data configured by the person running the project.
