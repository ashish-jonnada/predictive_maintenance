# Business Objective

## 1. Business Problem

Industrial rotating machinery can develop mechanical faults that may lead to degraded operation and unplanned production interruptions.

Traditional maintenance approaches may rely on fixed schedules or respond after a fault becomes apparent. A condition-monitoring system can provide earlier visibility into abnormal machine behavior and support maintenance personnel in deciding which equipment requires inspection.

The business problem addressed by this project is therefore:

> How can machine-condition data be converted into reliable, understandable health information that helps maintenance personnel identify equipment requiring attention before abnormal behavior contributes to an unexpected production interruption?

---

## 2. Primary Objective

> **Provide reliable and sufficiently persistent machine-condition warnings that help maintenance personnel identify equipment requiring inspection before abnormal behavior results in an unexpected production interruption.**

The system is intended to provide decision support, not to autonomously control or repair physical equipment.

---

## 3. Supporting Objectives

The system should support the following operational goals:

1. Identify known machine-condition/fault states from sensor data.
2. Detect deviations from learned baseline behavior.
3. Reduce unnecessary alerts by requiring sufficiently persistent evidence rather than reacting to a single noisy observation.
4. Provide understandable evidence behind a warning or predicted condition.
5. Help maintenance personnel prioritize machines/conditions requiring investigation.
6. Provide a foundation that can later be integrated with real-time machine-monitoring infrastructure.

---

## 4. Primary Stakeholder

### Maintenance Engineer

The primary user needs to answer:

- Which machine/condition needs attention?
- Is the machine behaving abnormally?
- What known fault condition does the behavior resemble?
- How confident is the system?
- Which measurements contributed to the warning?
- Should the machine be inspected?

The system should make these answers easier to obtain from sensor data.

---

## 5. Operational Decision Supported

The system supports the decision:

> **Should this machine/condition be investigated by maintenance personnel?**

A simplified decision flow is:

```text
Sensor observations
        |
        v
Machine-health analysis
        |
        +-------------------+
        |                   |
        v                   v
Known condition?       Abnormal vs baseline?
        |                   |
        +---------+---------+
                  |
                  v
          Persistent evidence?
                  |
            +-----+-----+
            |           |
           No          Yes
            |           |
            v           v
       Continue      Maintenance
       monitoring    investigation
```

The system does not directly command a repair.

---

## 6. Warning Philosophy

A single abnormal observation should not automatically become a maintenance alert.

Sensor measurements can contain:

- noise,
- temporary fluctuations,
- missing observations,
- irregular sampling intervals,
- operating-condition changes.

Therefore, the system should distinguish between:

### Observation

A single model/anomaly result.

### Warning

A sufficiently strong and persistent pattern that merits attention.

### Critical condition

A strong indication requiring prompt investigation, if the available evidence and validated thresholds justify such a level.

The exact persistence duration and alert thresholds will be determined later from data analysis, validation and operational assumptions. They will not be arbitrarily fixed at this stage.

---

## 7. Alert Severity Concept

The system may eventually represent machine health using a severity concept such as:

```text
GREEN  → Normal
YELLOW → Monitor
ORANGE → Warning
RED    → Critical
```

These labels are a product-design concept, not finalized numerical thresholds.

Final thresholds must be established using:

- model validation,
- anomaly-score behavior,
- class probabilities,
- persistence logic,
- false-alarm analysis,
- operational considerations.

---

## 8. False Positive vs False Negative

The two major operational errors are:

### False Positive

The system raises a warning when the machine does not require maintenance attention.

Too many false positives can cause:

- unnecessary inspections,
- wasted maintenance resources,
- alert fatigue,
- reduced trust in the system.

### False Negative

The system fails to warn when a genuine abnormal condition exists.

Potential consequences include:

- delayed inspection,
- increased risk of production interruption,
- additional maintenance cost,
- loss of confidence in the system.

The final model and alert strategy must therefore balance detection capability with unnecessary-alert control.

The exact cost of each error will depend on the real industrial environment. Since this project uses public test-bed data rather than a live factory maintenance-cost dataset, we will not invent monetary costs.

---

## 9. Business KPIs

Potential operational KPIs for a real deployment include:

### Reliability / Maintenance KPIs

- Unplanned machine downtime
- Machine availability
- Mean time between failures (MTBF)
- Mean time to repair (MTTR)
- Number of emergency maintenance events
- Maintenance response time

### Monitoring-System KPIs

- False alerts per machine/time period
- Missed critical conditions
- Warning lead time, when measurable
- Alert persistence rate
- User acceptance / alert trust

For the current portfolio project, real reductions in downtime or maintenance cost cannot be claimed because we do not have before/after operational data from a production factory.

---

## 10. ML Metrics vs Business KPIs

ML metrics measure predictive performance.

Examples:

- Precision
- Recall
- F1-score
- PR-AUC
- Confusion matrix
- Calibration, where appropriate
- Anomaly-detection performance measures

Business KPIs measure operational impact.

For example:

```text
ML metric:
Recall = how many relevant fault cases were detected

Business KPI:
How many potentially disruptive machine conditions were detected
early enough to support useful maintenance action
```

A high ML score does not automatically prove a reduction in factory downtime.

The project will keep these two levels separate.

---

## 11. What the System Will Not Claim

The current project will not claim that:

- the model guarantees prevention of machine failure;
- a detected fault means the machine will fail at a specific future time;
- the system can determine an exact failure time;
- the system can autonomously authorize or perform repairs;
- the project reduces factory downtime by a specific percentage;
- the public test-bed dataset represents proprietary production-factory data;
- the current dataset supports defensible RUL prediction.

RUL will only be considered if a genuine run-to-failure dataset with appropriate degradation trajectories and failure endpoints is introduced.

---

## 12. Success Definition

For the current project, business-level success means:

> The system produces reliable and understandable machine-condition information that can help a maintenance engineer identify abnormal or known fault conditions and prioritize equipment for inspection.

Technical model performance is necessary but not sufficient.

A useful system must also:

- avoid excessive false alarms;
- provide understandable outputs;
- behave consistently on time-dependent data;
- handle imperfect sensor data;
- be suitable for deployment as a monitoring service;
- maintain traceability between sensor observations, predictions and system versions.

---

## 13. Project Positioning

The project is positioned as:

> **An Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System**

It is a **decision-support system**, not an autonomous maintenance-control system.

The current core capabilities are:

1. Known-condition/fault classification.
2. Baseline-based anomaly detection.
3. Machine-health assessment and maintenance warning logic.

RUL prediction is a deferred capability pending suitable run-to-failure data.

---

## 14. Next Stage

**Part 1 → Step 3: ML Problem Definition**

The next stage will formally define:

- input data,
- target variables,
- prediction unit,
- prediction timing,
- classification formulation,
- anomaly-detection formulation,
- output probabilities/scores,
- validation constraints,
- and what constitutes a valid prediction.
