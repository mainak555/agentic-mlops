# Project Charter and Novelty

## Project Charter

### Purpose

Build and operate an automated predictive maintenance ML system that continuously converts dataset changes into deployable model and UI-ready schema artifacts.

### Technical Mission

- Train multiple candidate classifiers under reproducible pipeline execution.
- Select deployment candidate through explicit policy constraints and agentic reasoning.
- Publish model and schema artifacts to shared artifact stores.
- Serve inference through a dynamic UI that adapts to schema updates.

### Scope

In scope:
- Dataset registration to Hugging Face
- Train/test split automation
- Model training and evaluation with MLflow tracking
- Agentic model selection using Semantic Kernel plugin
- Model deployment packaging and registration
- Agentic schema generation for UI controls
- Streamlit inference UI deployment

Out of scope in current repository:
- Dedicated API gateway or microservice inference endpoint
- Online drift monitoring dashboards and alerting loops
- Automatic rollback/canary strategies

### Success Criteria

- Pipeline produces at least one selected model per valid run context.
- Deployed model and schema are available in Hugging Face model repo.
- UI can render form controls solely from schema metadata.
- Model input field changes can be reflected in UI without UI form code rewrite.

## High-Level Agentic MLOps Flow

```mermaid
flowchart TD
	A[Data Update<br/>data/engine_data.csv] --> B[Data Registration Workflow<br/>.github/workflows/data-reg-pipeline.yml]
	B --> C[Dataset Upload<br/>data-reg.py]
	C --> D[HF Dataset Repository]

	D --> X[External HF Webhook Relay Service<br/>separate repository/service]
	X --> E2[RepoDispatch Trigger<br/>repository_dispatch hf_webhook_event]

	E1[Direct Trigger<br/>push | workflow_dispatch] --> F[Run Mode Resolution<br/>.github/workflows/model-building-pipeline.yml]
	E2 --> F
	D --> G[Data Clean + Split<br/>model_building/train_test_split.py]
	F --> G

	G --> H[Model Build Entrypoint<br/>model_building/model_build.py]
	H --> I[Train + Evaluate + Log<br/>model_building/model_train.py]
	I --> J[Model Candidates<br/>model_building/model_config.py]

	J --> K[Selection Orchestrator<br/>model_building/model_select.py]
	K --> L[Selector Agent Runtime<br/>agents/model_selector_agent/run.py]
	L --> M[Policy + Output Schema<br/>agents/plugins/model_selector_plugin/select_model/skprompt.txt<br/>agents/plugins/model_selector_plugin/select_model/config.json]
	M --> N[Selected MLflow Run Tagged]

	N --> O[Model Deploy + Registry Tags<br/>model_building/model_deploy.py]
	O --> P[Schema Update Stage<br/>model_building/input_schema.py]
	P --> Q[Schema Agent Runtime<br/>agents/schema_generator_agent/run.py]
	Q --> R[Schema Contract + Prompt<br/>agents/plugins/schema_generator_plugin/generate_schema/config.json<br/>agents/plugins/schema_generator_plugin/generate_schema/skprompt.txt]

	O --> S[HF Model Repo + MLflow Registry]
	R --> S
	S --> T[UI Deploy/Restart Jobs<br/>.github/workflows/model-building-pipeline.yml]
	T --> U[Runtime Inference UI<br/>ui-app/app.py + ui-app/util.py]
```

Notes:
- The diagram reflects current workflow execution order where model deployment runs before input schema update.
- External data-driven pipeline triggering uses repository_dispatch hf_webhook_event through a separate webhook relay service.
- UI form rendering is schema-driven, so input field changes are propagated through artifacts rather than hardcoded UI form edits.

## Novelty: Agentic Dynamic Selection + Seamless UI Adaptation

## 1) Dynamic model selection without static winner logic

Instead of hardcoding a single metric sort, model_select.py builds candidate payloads and delegates final selection to an agent policy constrained by minimum thresholds for key metrics.

Outcome:
- Selection remains policy-driven and interpretable through justification tags.
- Candidate filtering can evolve without rewriting training logic.

## 2) Schema generation as deployment artifact

input_schema.py and schema_generator_agent/run.py transform selected feature context into a typed JSON schema. This schema is versioned and published with the model.

Outcome:
- Input contract is explicit and machine-readable.
- UI and model integration contract is artifact-based, not source-code coupled.

## 3) UI change without redevelopment cycle for feature drift-driven input changes

ui-app/app.py renders controls by iterating schema inputs and field types. When a new model version introduces a changed feature set, updated schema can drive UI controls automatically.

Outcome:
- No recurring manual edit cycle for adding/removing form fields after each model/schema update.
- Faster deployment response when data changes force feature set updates.

## 4) Agentic orchestration integrated with MLOps lifecycle

Model selection and schema generation agents are inserted as pipeline stages, not side utilities.

Outcome:
- Decision intelligence and interface contract generation become first-class deployment steps.
- The project demonstrates an applied agentic MLOps pattern rather than standalone model training.
