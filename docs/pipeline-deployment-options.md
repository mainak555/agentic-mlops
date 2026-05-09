# Pipeline Deployment Options

## Summary

The repository supports multiple deployment pathways based on the change source and desired operational speed.

## Option 1: Full pipeline execution

When to use:
- New data is available and full retraining is required.
- Model code changes require full rebuild and redeployment.

Execution path (workflow mode Full):
1. Data ingestion check and split processing
2. Model training
3. Agentic model selection
4. Model deployment to Hugging Face model repo
5. Input schema generation and upload
6. UI deployment to Hugging Face Space

## Option 2: Partial (manual workflow_dispatch)

When to use:
- You want a controlled rerun without data_clean.
- You want retrain/deploy plus service restart behavior from manual trigger.

Execution path:
- setup -> model_build -> model_deploy -> ui_restart

## Option 3: RepoDispatch (external data event)

When to use:
- An external Hugging Face data-change event should trigger retraining automatically.
- You are using a webhook relay that emits repository_dispatch hf_webhook_event.

Execution path:
- setup -> data_clean -> model_build -> model_deploy -> ui_restart

Dependency:
- Requires an external webhook relay service (outside this repository) to send repository_dispatch events to GitHub Actions.

## Option 4: UI-only deployment

When to use:
- Changes are restricted to ui-app components.
- No model rebuild required.

Execution path (workflow mode UI Only):
- Skip training and model deployment jobs.
- Deploy updated UI package to Hugging Face Space.

## Option 5: Script-driven manual deployment

When to use:
- Local development or ad hoc operations.
- Debugging specific stage without full CI orchestration.

Typical scripts:
- data-reg.py
- model-reg.py
- app-reg.py
- model_building/model_build.py
- model_building/model_select.py
- model_building/model_deploy.py
- model_building/input_schema.py

## Decision Matrix

- Data file change in data/engine_data.csv:
  - Use data registration workflow, then trigger model pipeline via RepoDispatch (external webhook relay).
- Training logic/model config change:
  - Use full pipeline execution.
- Schema agent/plugin change:
  - Use model deployment chain including input schema stage.
- UI styling/layout-only change:
  - Use UI-only deployment.

## Deployment Artifacts Produced

- Dataset repo artifacts: source CSV and train/test CSV.
- Model repo artifacts: serialized model and input_schema.json.
- MLflow artifacts and tags: run metrics, top-k features, deployment tags, registry metadata.
- Space repo artifacts: Streamlit app package.

## Constraints to Consider

- Inference serving is UI-based, not API-service based.
- Drift monitoring and alerting are not part of deployment option logic.
- Rollback strategy must be handled operationally outside the current workflows.
