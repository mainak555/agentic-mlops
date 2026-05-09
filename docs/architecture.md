# Architecture

## System Overview

The repository implements a staged MLOps architecture for binary engine maintenance prediction.

High-level layers:
1. Data and dataset registration
2. Training and evaluation
3. Agentic model selection
4. Model deployment and schema generation
5. UI inference serving

## Component Map

- Data registration: data-reg.py
- Dataset split and cleanup: model_building/train_test_split.py
- Multi-model training and metrics: model_building/model_train.py
- Candidate model selection by agent: model_building/model_select.py
- Selected model packaging and publish: model_building/model_deploy.py
- Input schema generation by agent: model_building/input_schema.py
- Schema generation runtime: agents/schema_generator_agent/run.py
- Model selector runtime: agents/model_selector_agent/run.py
- Streamlit inference app: ui-app/app.py
- Runtime artifact fetch: ui-app/util.py

## End-to-End Runtime Flow

1. Data file is uploaded to Hugging Face dataset repo by data-reg.py.
2. train_test_split.py loads dataset from Hugging Face, removes Unnamed columns, and creates stratified train/test split.
3. model_train.py trains configured estimators using preprocessing, outlier capping, hyperparameter search, and threshold optimization.
4. MLflow logs runs, metrics, features, and run tags including pipeline run id.
5. model_select.py queries candidate runs for the current pipeline run and invokes the selector agent.
6. Selected run is tagged selected_for_deployment.
7. model_deploy.py retrains the selected model on selected feature subset, serializes .joblib, uploads artifact to Hugging Face model repo, and writes model registry metadata.
8. input_schema.py invokes schema generator agent and publishes input_schema.json to Hugging Face model repo.
9. ui-app/app.py loads model + schema at runtime and renders dynamic form fields from schema definitions.

## Agentic Subsystems

### Model Selector Agent

Source integration:
- model_building/model_select.py
- agents/model_selector_agent/run.py
- agents/plugins/model_selector_plugin/select_model/skprompt.txt

Behavior summary:
- Builds candidate payload from MLflow runs in current pipeline execution.
- Applies explicit constraints via agent payload: min_f1, min_recall, min_precision.
- Selects one mlflow run id and records justification.

### Schema Generator Agent

Source integration:
- model_building/input_schema.py
- agents/schema_generator_agent/run.py
- agents/plugins/schema_generator_plugin/generate_schema/skprompt.txt

Behavior summary:
- Inspects selected input features from deployed model context.
- Derives categorical, numeric, and text feature descriptors.
- Generates JSON schema validated against plugin schema config.
- Drives UI rendering without hardcoded form field definitions.

## Artifact Contracts

### MLflow

Per run tags and metrics include:
- pipeline_run_id
- model_name
- test_f1, test_recall, test_precision, test_roc_auc
- decision_threshold
- inference_latency_ms
- model_complexity

MLflow artifacts include:
- feature_analysis/top_k_features.json
- feature_schema/raw_features.json
- hf_model/metadata.json (post deployment)

### Hugging Face

Dataset repo stores:
- engine_data.csv
- train.csv
- test.csv

Model repo stores:
- experiment_name.joblib
- input_schema.json

Space repo stores:
- Streamlit app and runtime files

## UI Decoupling Design

The UI form is schema-driven. ui-app/app.py iterates over schema inputs and renders controls by field type and options. Because the UI does not hardcode model input fields, schema updates from new model feature sets can be consumed without rewriting UI form code.

This is the key mechanism that removes repeated UI redevelopment for model input changes.
