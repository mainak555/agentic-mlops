# Predictive Maintenance Technical Documentation

This documentation set describes the technical architecture, agentic model selection behavior, deployment pathways, and current data drift handling implemented in this repository.

## Repository Description

This project is an agent-driven MLOps system for predictive maintenance classification. It trains multiple candidate models, evaluates them in MLflow, uses an AI selector agent to choose the deployment candidate under metric constraints, deploys the selected model artifact to Hugging Face, generates input schema with a schema agent, and serves predictions through a schema-driven Streamlit UI.

Core design intent:
- Keep model training, model selection, schema generation, and UI serving as separate stages.
- Enable dynamic model and schema updates without UI source-code rewrites.
- Use pipeline modes so teams can run full retraining or UI-only deployment based on change type.

## Document Map

- [Architecture](architecture.md)
- [Deployment and Development Guide](deployment-development.md)
- [Project Charter and Novelty](project-charter-novelty.md)
- [Data Drift Handling](data-drift-handling.md)
- [Pipeline Deployment Options](pipeline-deployment-options.md)

## External Dependencies (Operational)

- External Hugging Face webhook relay service is required to emit repository_dispatch hf_webhook_event for automatic data-event retraining path.
