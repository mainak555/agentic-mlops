# Data Drift Handling (Current Implementation)

This document describes what is currently implemented for data drift related robustness and what is not currently implemented.

## Implemented Safeguards

## 1) Input cleanup before split

In model_building/train_test_split.py:
- Removes columns with names starting with Unnamed.
- Performs stratified train/test split on target variable.

This improves data consistency for downstream training.

## 2) Outlier clipping in preprocessing

In model_building/util2.py and model_building/model_train.py:
- IQRCapper computes per-feature lower and upper bounds using IQR factor 1.5.
- Numeric values are clipped to these bounds during pipeline preprocessing.

This reduces sensitivity to extreme values in numeric sensors.

## 3) Threshold-aware classification behavior

In model_building/model_train.py:
- Decision threshold is selected from precision-recall curve targeting recall constraint.
- Threshold is logged and carried to deployed schema context.

This keeps decision behavior aligned with recall-oriented objective under changing distributions.

## 4) Feature contract checks for schema generation

In agents/schema_generator_agent/run.py:
- Schema generation directly indexes expected feature columns from dataframe context.
- Missing feature fields cause immediate failure rather than silent mismatch.

This prevents deployment of a schema that does not align with expected feature inputs.

## 5) Data change retraining trigger path

In .github/workflows/data-reg-pipeline.yml and .github/workflows/model-building-pipeline.yml:
- Dataset updates can trigger registration and subsequent model rebuild/deploy flows.

This enables batch refresh when source data changes.

## Not Implemented in Current Codebase

- No explicit statistical drift detectors (for example, PSI, KS, Wasserstein).
- No online concept drift detection tied to real-world labels.
- No production inference logging loop for continuous drift diagnostics.
- No automatic alerting based on drift thresholds.
- No automatic rollback mechanism based on drift signals.

## Practical Interpretation

Current handling is best described as batch retraining readiness plus preprocessing robustness, not full drift observability. The system can absorb new datasets through automated pipeline runs, but it does not currently compute, track, and alert on drift metrics during live operations.
