# Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System

## 1. Problem Understanding

### 1.1 Real-World Problem

Industrial rotating machinery can develop mechanical faults that affect production reliability and may eventually lead to unplanned downtime.

The objective of this project is to build a machine-health monitoring system that uses sensor measurements to identify abnormal machine behavior and recognize known fault conditions early enough to support maintenance inspection and planning.

The system is intended to support maintenance decisions; it is not intended to replace a qualified maintenance engineer.

---

## 2. Equipment / Use Case

### Equipment Domain

**Industrial rotating machinery, represented by an electric-motor test bed.**

The current project uses motor condition-monitoring data containing vibration, temperature, speed and voltage-related measurements.

### Current Use Case

Given recent machine-condition measurements, the system should:

1. Determine whether the machine resembles its normal/baseline condition.
2. Identify a known fault condition when sufficient evidence exists.
3. Detect behavior that is abnormal relative to the learned baseline.
4. Present the resulting machine-health information in a form useful for maintenance inspection.

---

## 3. Stakeholders

### Primary Stakeholder — Maintenance Engineer

Needs to understand:

- Which machine/condition needs attention?
- Is the behavior abnormal?
- What fault condition does it resemble?
- How confident is the system?
- Which sensor signals contributed to the warning?

### Secondary Stakeholder — Production / Operations Manager

Needs visibility into:

- Machine health
- Potential production interruptions
- Fault trends
- Maintenance-priority signals

### Technical Stakeholder — ML / System Engineer

Needs to manage:

- Data quality
- Model performance
- Inference reliability
- Model/data drift
- Model versions
- Retraining

---

## 4. Proposed System

High-level flow:

```text
Machine Sensors
      |
      v
Data Ingestion
      |
      v
Data Validation
      |
      v
Data Processing
      |
      v
Feature Engineering
      |
      +----------------------+
      |                      |
      v                      v
Fault / Condition       Anomaly Detection
Classification
      |                      |
      +----------+-----------+
                 |
                 v
          Machine Health
                 |
                 v
        Maintenance Support
```

---

## 5. Current Data Inputs

The currently selected CSV contains measurements associated with:

- Four thermocouple channels
- Four accelerometer channels
- Tachometer speed
- Voltage
- Timestamp
- Machine-condition status

The accelerometer measurements include derived vibration metrics such as RMS, peak and related signal measures.

The exact usable feature set will be finalized during the Data Requirements, Data Validation and EDA stages.

We will not assume that every available column should be used for modelling.

---

## 6. Current Condition Labels

The uploaded `final.csv` contains four observed condition labels:

- `baseline`
- `imbalance`
- `looseness`
- `angular_misalignment`

These labels are the basis for the initial supervised condition/fault-classification problem.

The label distribution and temporal organization will be investigated formally during data validation and EDA.

---

## 7. ML Capabilities

### 7.1 Fault / Condition Classification

Primary supervised task:

> Given machine sensor observations, classify the observed machine condition among the supported classes.

Current target:

```text
baseline
imbalance
looseness
angular_misalignment
```

This is a multi-class classification problem.

### 7.2 Anomaly Detection

A separate anomaly-detection capability will learn the characteristics of baseline/normal behavior and determine whether new observations deviate sufficiently from that behavior.

Classification and anomaly detection will remain conceptually separate:

- Classification asks: **Which known condition does this resemble?**
- Anomaly detection asks: **Does this look abnormal compared with normal behavior?**

### 7.3 RUL — Explicitly Out of Current Scope

Remaining Useful Life (RUL) prediction is **not supported by the current CSV as a defensible primary task**.

The current dataset represents different machine-condition states rather than a clearly documented run-to-failure degradation trajectory with a known failure endpoint.

Therefore, we will not manufacture RUL labels from this data.

RUL may be added later only if a genuine run-to-failure dataset with appropriate degradation trajectories and failure endpoints is obtained.

---

## 8. Important Data Limitations Already Identified

### 8.1 Irregular timestamps

The observations are predominantly around minute-level intervals but contain irregular gaps.

Therefore, the project must not assume that every row represents an exactly fixed time interval.

### 8.2 Missing measurements

Some sensor columns contain missing values.

At least one tachometer-related column is completely missing in the current CSV.

A completely missing feature will not be artificially imputed without first understanding why it exists.

### 8.3 Temporal dependence

Observations are time-dependent and condition measurements occur in contiguous periods.

A naive random row-level train/test split may produce overly optimistic results because nearby observations from the same operating period can appear in both training and testing data.

The final validation strategy must account for temporal structure and experimental grouping.

### 8.4 Potential operating-condition confounding

Voltage and other operating conditions may be associated with particular fault states.

The model must therefore be evaluated to determine whether it is learning genuine machine-condition signatures or simply exploiting experimental/operating-condition differences.

---

## 9. Data Provenance

### Current Dataset

The current `final.csv` is the **provided dataset file from the Cutsforth InsightCM portion of the UTK-ASL predictive-maintenance dataset repository**.

It was not generated by this project.

The project will preserve the original file as raw/source data and will create separate processed datasets after validation and transformation.

### Official Source

UTK-ASL — Dataset: Digital Twin / Predictive Maintenance

https://github.com/UTK-ASL/Dataset_digital_twin_predictive_maintenance

---

## 10. Real vs Simulated Data Statement

The project documentation will clearly distinguish between:

- publicly provided experimental/test-bed data,
- processed data created by this project,
- simulated streaming/replay during deployment,
- and any future real industrial data.

We will not claim that the current dataset is proprietary factory production data.

---

## 11. Non-Goals

The initial version will not:

- claim exact future failure time without appropriate run-to-failure data;
- generate artificial RUL labels solely to add an RUL feature;
- automatically command physical machine repairs;
- treat model predictions as a replacement for engineering inspection;
- optimize for accuracy alone while ignoring leakage and deployment behavior.

---

## 12. Success Definition for This Stage

Problem Understanding is considered complete when the project can clearly answer:

> **What problem are we solving?**

Answer:

> We are building a condition-monitoring and predictive-maintenance support system for industrial rotating machinery that uses machine-condition sensor data to detect abnormal behavior and identify known mechanical fault conditions, providing actionable health information for maintenance inspection.

The exact business KPIs, ML metrics, prediction thresholds and operational success criteria will be defined in the next project stages.

---

## 13. Current Decision

### Core Project

**Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System**

### Core ML Tasks

1. Fault / condition classification
2. Anomaly detection

### Deferred Capability

3. RUL prediction — only if suitable run-to-failure data is obtained

---

## 14. Next Stage

**Part 1 → Step 2: Business Objective**

The next stage will translate the technical problem into measurable business/operational objectives, including:

- what decision the system supports;
- how early a warning is useful;
- what constitutes a useful alert;
- false-positive vs false-negative consequences;
- measurable success criteria.
