# System Architecture

## 1. System Goal

The system is an **Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System**.

Its purpose is to transform machine-condition data into useful health information that can support maintenance personnel.

The system is designed as decision support.

It does not autonomously repair, control, or shut down physical machinery.

---

# 2. High-Level Architecture

```text
                    INDUSTRIAL MACHINE
                           │
                     Sensors / DAQ
                           │
                           ▼
                  DATA INGESTION LAYER
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       Historical Data            Simulated Live
                                  Data Replay
              │                         │
              └────────────┬────────────┘
                           ▼
                  DATA VALIDATION
                           │
                           ▼
               WINDOW / FEATURE PIPELINE
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       FAULT CLASSIFIER          ANOMALY DETECTOR
              │                         │
              │                         │
              └────────────┬────────────┘
                           ▼
                  MACHINE HEALTH ENGINE
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
            Health Status       Explanation
                 │                   │
                 └─────────┬─────────┘
                           ▼
                  MAINTENANCE ALERT
                           │
                           ▼
                       REST API
                           │
                           ▼
                       DASHBOARD
                           │
                           ▼
                   HUMAN DECISION
                           │
                           ▼
                 MAINTENANCE ACTION
```

---

# 3. Operational Data Flow

## 3.1 Development

During development, historical data is used:

```text
Historical data
      ↓
Validation
      ↓
Processing
      ↓
Feature engineering
      ↓
Model training
      ↓
Evaluation
      ↓
Model artifact
```

---

## 3.2 Deployment Demonstration

Because the project does not have a physical factory machine connected to it, the deployed system will use historical sensor data as a **simulated live stream**.

```text
Historical sensor records
          ↓
      Data Replay
          ↓
    Ingestion/API
          ↓
 Validation + Windowing
          ↓
      ML Inference
          ↓
   Health Assessment
          ↓
       Dashboard
```

This is explicitly a simulation of live ingestion, not a claim of a real factory connection.

---

# 4. Manual Input

A manual/API test-input mechanism may exist during development and debugging.

Example:

```text
Manual test payload
        ↓
      REST API
        ↓
       Model
        ↓
    Prediction
```

However:

> **Manual sensor entry is not the primary production workflow.**

In a real deployment, sensor/DAQ/gateway infrastructure would provide the measurements automatically.

---

# 5. Multi-Machine Design

The architecture is designed to support multiple machines.

Conceptually:

```text
M001 ── sensors ──┐
M002 ── sensors ──┤
M003 ── sensors ──┼──→ Data Ingestion
M004 ── sensors ──┤
...               ┘
                         ↓
                   ML Inference
                         ↓
                   Health Engine
                         ↓
                     Dashboard
```

The system should carry a machine identifier through the pipeline.

Example:

```text
machine_id
timestamp
sensor measurements
```

The current project data represents experimental/testbed conditions rather than a real production fleet, so the architecture should support multiple machines without falsely claiming that the current dataset contains a large industrial fleet.

---

# 6. ML Layer

The ML layer contains two logically separate components.

## 6.1 Fault / Condition Classifier

Input:

```text
Validated recent sensor window
```

Output:

```text
Predicted known condition
+
class scores/probabilities
```

---

## 6.2 Anomaly Detector

Input:

```text
Validated recent sensor window
```

Output:

```text
Anomaly score
```

The anomaly score is not automatically a failure probability.

---

# 7. Machine Health Layer

The health layer sits between ML inference and the user interface.

It interprets model outputs according to validated rules.

Example:

```text
Classifier:
Faulted bearing

Anomaly detector:
High anomaly score

        ↓

Health assessment:
Elevated concern

        ↓

Maintenance support:
Inspection recommended
```

The health layer should not silently convert a model prediction into an unsupported claim of certain failure.

---

# 8. API Layer

The REST API provides a stable interface between the inference system and applications.

Conceptually:

```text
Client / Replay
      ↓
POST /predict
      ↓
Input validation
      ↓
Feature/window processing
      ↓
Model inference
      ↓
Health assessment
      ↓
JSON response
```

A future response may contain:

```text
machine_id
timestamp
predicted_condition
class_scores
anomaly_score
health_status
supporting_evidence
```

The exact API contract will be defined during implementation.

---

# 9. Dashboard Layer

The dashboard represents the maintenance-facing view.

A high-level view may contain:

```text
Factory / Machine Overview
        ↓
Machine status
        ↓
Selected machine
        ↓
Current condition
        ↓
Anomaly score
        ↓
Sensor trends
        ↓
Supporting evidence
        ↓
Maintenance attention
```

The dashboard is not the ML model.

Its purpose is to make model outputs operationally understandable.

---

# 10. ML Lifecycle Architecture

Operational inference and model development are connected through an ML lifecycle.

```text
Data
  ↓
Validation
  ↓
Training Dataset
  ↓
Experiment
  ↓
Model Evaluation
  ↓
Model Version
  ↓
Deployment
  ↓
Inference
  ↓
Monitoring
  ↓
Drift / Performance Signals
  ↓
Retraining Decision
  ↓
New Model Version
```

---

# 11. Monitoring

The deployed system should eventually monitor:

### Data quality

- missing values;
- schema changes;
- invalid values;
- unexpected distributions.

### Data drift

Changes in sensor distributions compared with the training/reference distribution.

### Prediction behavior

- class distribution;
- anomaly-score behavior;
- confidence behavior.

### System health

- API latency;
- error rate;
- service availability.

Where ground-truth labels become available later, model performance can also be monitored.

---

# 12. Deployment Architecture

The intended V1 deployment path is:

```text
Model
  ↓
Inference service
  ↓
REST API
  ↓
Docker container
  ↓
Cloud / server deployment
  ↓
Dashboard
```

The exact cloud provider and infrastructure choices will be decided during the deployment stage.

We will not add infrastructure complexity that does not serve a project requirement.

---

# 13. Security and Reliability Boundaries

The system should validate incoming requests before inference.

Important boundaries include:

- input schema validation;
- model-version identification;
- safe failure behavior;
- logging;
- separation between prediction and physical machine control.

The project does not authorize the ML service to directly control industrial equipment.

---

# 14. Architecture Principle

The project is not:

```text
Dataset
   ↓
Notebook
   ↓
Accuracy
```

It is:

```text
Business Problem
      ↓
Data Strategy
      ↓
Data Engineering
      ↓
ML
      ↓
Inference
      ↓
Health Assessment
      ↓
API
      ↓
Dashboard
      ↓
Deployment
      ↓
Monitoring
```

This architecture keeps the ML model as one component of a larger engineering system.
