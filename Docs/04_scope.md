# Project Scope — V1

## 1. Scope Purpose

This document freezes the boundary of Version 1 of the Predictive Maintenance project.

The purpose is to prevent uncontrolled expansion and to make sure every component has a clear reason to exist.

---

# 2. V1 Project

## Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System

The system will analyze rotating-machinery condition data and provide:

- known-condition/fault classification;
- baseline-based anomaly detection;
- machine-health assessment;
- maintenance-oriented alerts/supporting evidence;
- API-based inference;
- dashboard visualization;
- containerized deployment;
- basic ML-system monitoring.

---

# 3. In Scope

## 3.1 Problem and Requirements

We will document:

- business problem;
- users;
- operational decisions;
- ML problem definition;
- data requirements;
- system boundaries.

---

## 3.2 Data Engineering

V1 includes:

- raw-data organization;
- schema validation;
- timestamp validation;
- missing-value analysis;
- duplicate/repeated-observation analysis;
- sensor sanity checks;
- label validation;
- experiment-boundary analysis;
- leakage analysis;
- controlled preprocessing;
- reproducible feature generation.

Raw source files remain unchanged.

---

## 3.3 Fault / Condition Classification

V1 includes a supervised multi-class classifier for the known conditions represented by the selected System1 modelling data.

The current known conditions are:

```text
baseline
imbalance
eccentric_rotor
bent_shaft
faulted_bearing
faulted_coupling
```

The final modelling feature set and windowing strategy will be determined after data validation and EDA.

---

## 3.4 Anomaly Detection

V1 includes a separate baseline-oriented anomaly-detection component.

Its purpose is:

> Detect behavior that differs substantially from learned baseline behavior.

The anomaly detector will not be presented as an exact failure predictor.

---

## 3.5 Machine Health Assessment

V1 will contain a non-ML health/alert layer that combines validated model outputs into understandable machine-health information.

Example:

```text
Condition prediction
+
Anomaly score
+
Confidence/evidence
        ↓
Health status
        ↓
Maintenance attention
```

The exact thresholds will be established after model validation.

---

## 3.6 Explainability

Where appropriate, V1 will provide evidence explaining the model result.

Possible evidence includes:

- influential sensor features;
- sensor trends;
- condition scores;
- anomaly behavior.

The final explanation method will depend on the selected model.

---

## 3.7 API

V1 will expose model inference through a REST API.

The API will support:

- input validation;
- model inference;
- health assessment;
- structured JSON responses;
- error handling;
- logging.

---

## 3.8 Dashboard

V1 will provide a maintenance-oriented dashboard showing:

- machine identifier;
- current health status;
- predicted condition;
- confidence/scores;
- anomaly score;
- relevant sensor trends;
- supporting evidence;
- maintenance attention recommendation.

---

## 3.9 Simulated Live Inference

Because the project does not have a physical machine connected to the deployed system, V1 will demonstrate live-like behavior using **historical sensor-data replay**.

```text
Historical records
       ↓
Replay
       ↓
Ingestion
       ↓
Inference
       ↓
Dashboard update
```

This is a simulation and will be described honestly as such.

A manual/API test-input mechanism may also exist for development and debugging, but manual sensor entry is not the primary production architecture.

---

## 3.10 Deployment

V1 includes:

```text
Model
 ↓
Inference service
 ↓
REST API
 ↓
Docker
 ↓
Cloud/server deployment
```

The exact platform will be selected later based on practicality and project requirements.

---

## 3.11 MLOps / Monitoring

V1 will demonstrate the ML lifecycle beyond model training.

Planned areas include:

- experiment tracking;
- model versioning;
- reproducible inference;
- basic data-quality monitoring;
- drift monitoring where appropriate;
- API/service monitoring;
- retraining workflow design.

The project will not attempt to reproduce a full enterprise MLOps platform.

---

# 4. Data Scope

The project uses the available Bently Nevada System1 and Cutsforth InsightCM sources for different purposes.

### System1

Primary role:

```text
Known-condition / fault modelling
```

### InsightCM

Complementary role where justified:

```text
Dense time-series analysis
Anomaly / monitoring experiments
Raw ingestion understanding
Historical replay
```

### Original experimental files

Used for:

```text
Provenance
Experiment-boundary understanding
Validation
```

### Raw JSON

Used where appropriate for:

```text
Raw ingestion understanding
Historical replay / simulated streaming
```

Data sources will not be blindly concatenated.

---

# 5. RUL Scope

## Excluded from V1

RUL (Remaining Useful Life) is **not included in V1**.

Reason:

The current data does not provide a sufficiently documented run-to-failure lifecycle with a reliable failure/end-of-life target for defensible RUL modelling.

We will not create artificial RUL labels from fault classes.

### Future extension

If a suitable run-to-failure dataset is deliberately introduced later, RUL can be added as a separate modelling capability.

---

# 6. Explicitly Out of Scope

V1 will not include:

- autonomous machine shutdown;
- PLC/industrial-control commands;
- automatic physical repair;
- guaranteed failure prevention;
- exact future failure-time prediction;
- unsupported RUL estimation;
- unsupported production downtime-reduction claims;
- blindly combining incompatible datasets;
- one separate ML model for every machine without evidence that this is required;
- unnecessary enterprise infrastructure such as a large Kubernetes platform solely for demonstration.

---

# 7. Human-in-the-Loop Boundary

The system provides information to maintenance personnel.

```text
Machine
   ↓
Sensors
   ↓
ML system
   ↓
Health assessment
   ↓
Maintenance support
   ↓
Human inspection/decision
```

The human remains responsible for the physical maintenance decision.

---

# 8. V1 Success Criteria

V1 is successful when we can demonstrate an end-to-end, reproducible workflow:

```text
Raw data
   ↓
Validated data
   ↓
Feature pipeline
   ↓
Classification
   +
Anomaly detection
   ↓
Evaluation
   ↓
Health assessment
   ↓
API
   ↓
Dashboard
   ↓
Simulated live replay
   ↓
Docker deployment
   ↓
Monitoring
```

Success will be demonstrated using technically valid evaluation rather than a single accuracy number.

---

# 9. Portfolio / Interview Demonstration

The final project should allow us to explain:

1. Why the business needs condition monitoring.
2. Why the ML problems were chosen.
3. How the data was selected and validated.
4. How leakage was prevented.
5. How the model was evaluated.
6. Why the final model was selected.
7. How anomaly detection complements classification.
8. How predictions become maintenance-oriented information.
9. How the model is exposed through an API.
10. How historical data is replayed as simulated live data.
11. How the system is containerized and deployed.
12. How the deployed ML system can be monitored and retrained.

---

# 10. V1 Boundary Summary

```text
                    V1
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    DATA          ML            SYSTEM
       │             │             │
       │       ┌─────┴─────┐       │
       │       ↓           ↓       │
       │  Classification  Anomaly │
       │       │           │       │
       │       └─────┬─────┘       │
       │             ↓             │
       │       Health Assessment   │
       │             ↓             │
       └─────────────┼─────────────┘
                     ↓
                    API
                     ↓
                 Dashboard
                     ↓
             Simulated Live Replay
                     ↓
                  Docker
                     ↓
                Deployment
                     ↓
                 Monitoring
```

This is the V1 boundary. New capabilities will only be added after a clear technical or business justification.
