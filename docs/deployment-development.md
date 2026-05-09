# Deployment and Development Guide

## Deployment Topologies Supported

1. Local script-driven execution for development.
2. GitHub Actions orchestrated training and deployment pipeline.
3. Hugging Face hosted deployment for dataset, model artifacts, and Streamlit app space.

## Core Entrypoints

- data-reg.py: Creates dataset repo and uploads engine_data.csv.
- model-reg.py: Creates model repo.
- app-reg.py: Creates space repo, sets app variables, uploads ui-app folder.
- model_building/model_build.py: Model training job.
- model_building/model_select.py: Agentic model selection job.
- model_building/model_deploy.py: Deployment artifact publish job.
- model_building/input_schema.py: Schema generation and upload job.
- ui-app/app.py: Inference UI entrypoint.

## Required Environment Variables

Common variables used by scripts and workflows:
- HF_TOKEN
- HF_REPO
- MLFLOW_TRACKING_URI
- MLFLOW_EXPERIMENT_NAME
- MLFLOW_TRACKING_USERNAME
- MLFLOW_TRACKING_PASSWORD
- GITHUB_RUN_ID (pipeline context)
- AZURE_OPENAI_DEPLOYMENT
- AZURE_OPENAI_ENDPOINT
- AZURE_OPENAI_API_KEY

Optional local controls:
- USE_OPENAI, OPENAI_MODEL, OPENAI_API_KEY
- MODEL_REPO, MODEL_FILE
- TARGET_VARIABLE
- TARGET_RECALL
- DECISION_THRESHOLD

## Local Development Flow

Typical sequence:
1. Register data/model/app repos as needed.
2. Run train/test split generation.
3. Run model training.
4. Run model selection.
5. Run model deployment.
6. Run input schema generation.
7. Run Streamlit UI.

Example command sequence:
- python data-reg.py
- python model-reg.py
- python model_building/train_test_split.py
- python model_building/model_build.py
- python -m model_building.model_select
- python model_building/model_deploy.py
- python -m model_building.input_schema
- streamlit run ui-app/app.py --server.port=7860 --server.address=0.0.0.0

## CI/CD Workflow Modes

Defined in .github/workflows/model-building-pipeline.yml:

- Full:
  - Manual workflow_dispatch mode set to Full.
  - Executes data_clean, model_build, model_deploy, and ui_deploy.
- Partial:
  - Manual workflow_dispatch mode set to Partial.
  - Skips data_clean, runs model_build and model_deploy, then ui_restart.
- Push:
  - Triggered by push changes under model_building or ui-app.
  - If changes are only under ui-app, run_mode becomes UI Only.
  - Otherwise run_mode becomes Push and executes model_build, model_deploy, and ui_deploy.
- UI Only:
  - Triggered when push changes are limited to ui-app.
  - Skips training and model deployment, runs ui_deploy only.
- RepoDispatch:
  - Triggered by repository_dispatch event type hf_webhook_event.
  - Executes data_clean, model_build, model_deploy, and ui_restart.

## External Webhook Relay Dependency

The RepoDispatch path depends on an external Hugging Face webhook relay service that emits GitHub repository_dispatch events (event type hf_webhook_event). This relay is not implemented in this repository but is required for automatic pipeline triggering from external dataset-change events.

Data registration workflow:
- .github/workflows/data-reg-pipeline.yml uploads data/engine_data.csv to dataset repo on push.

## Runtime Serving Path

At app startup, ui-app/util.py downloads model artifact and input schema from Hugging Face model repo. The app uses schema metadata to render form fields and runs inference with model.predict_proba and deployed decision threshold.

## Operational Notes

- The implementation is strong for automated retraining and schema refresh workflows.
- There is no dedicated REST inference service in this repository; serving path is Streamlit-first.
- Production observability, alerting, and rollback orchestration are limited and should be managed externally if required.
