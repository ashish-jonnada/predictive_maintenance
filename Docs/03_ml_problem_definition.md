# ML Problem Definition

## 1. Purpose

This document converts the business objective into clear machine-learning problems.

Our project is an **Industrial Rotating Machinery Condition Monitoring & Predictive Maintenance Support System**.

The ML system supports maintenance decisions. It does not automatically control or repair physical machinery.

---

## 2. The Two ML Problems

The current project has two separate ML tasks:

1. **Fault / Condition Classification**
2. **Baseline-based Anomaly Detection**

They answer different questions:

```text
Classification:
"Which known machine condition does this behavior resemble?"

Anomaly Detection:
"Does this behavior look sufficiently different from normal behavior?"
```

---

# 3. ML Problem 1 — Fault / Condition Classification

## 3.1 Problem Type

This is a **supervised multi-class classification** problem.

"Multi-class" simply means that the model chooses between several possible classes.

For the current Bently Nevada System1 data, the observed classes are:

```text
baseline
imbalance
eccentric_rotor
bent_shaft
faulted_bearing
faulted_coupling
```

These labels describe the conditions represented in the dataset. They do not mean that the machine is guaranteed to fail in the future.

---

## 3.2 What Goes Into the Model?

The input is commonly called **X**.

Candidate inputs include validated vibration measurements and derived vibration metrics from the available channels.

Examples:

- bias;
- derived peak;
- direct measurement;
- direct RMS;
- velocity peak;
- velocity RMS;
- justified time-related information;
- operating-condition information only after leakage/confounding analysis.

The final feature list is **not frozen yet**.

It will be decided after:

1. data validation;
2. EDA;
3. feature engineering;
4. leakage and confounding checks.

Having a column does not automatically mean that it should be used as a model feature.

---

## 3.3 What Does the Model Predict?

The answer is called **y**, or the target.

```text
y ∈ {
    baseline,
    imbalance,
    eccentric_rotor,
    bent_shaft,
    faulted_bearing,
    faulted_coupling
}
```

Example:

```text
Sensor measurements
        |
        v
      Model
        |
        v
Predicted condition:
faulted_bearing
```

The model may also return class probabilities/scores.

We will later check whether those probabilities are properly calibrated before interpreting them as meaningful probabilities.

---

# 4. What Is One Prediction?

This is an important design decision.

Our data is time-dependent. It may contain several related measurements around the same period.

Therefore, we should not automatically assume:

```text
one raw row = one independent prediction
```

For the operational system, we will investigate using a **recent observation window**.

Conceptually:

```text
t1   t2   t3   t4   t5
 |    |    |    |    |
 +----+----+----+----+
          |
          v
    Recent time window
          |
          v
       ML model
          |
          v
   Machine condition
```

The exact window length, overlap and aggregation method will be decided after studying the actual sampling structure and fault periods.

We will not arbitrarily choose a window length before examining the data.

---

# 5. ML Problem 2 — Anomaly Detection

## 5.1 Problem Type

This is **baseline-based anomaly detection**.

The question is:

> **Does the current machine behavior look sufficiently different from learned normal/baseline behavior?**

This is different from classification.

```text
Classification:
"Which known fault is this?"

Anomaly detection:
"Does this look abnormal compared with normal?"
```

---

## 5.2 How It Works

Conceptually:

```text
Baseline / normal data
          |
          v
 Learn normal behavior
          |
          v
New machine observation/window
          |
          v
    Anomaly score
          |
          v
Normal / Anomalous
```

A low or high score must be interpreted according to the chosen algorithm.

Therefore:

> **Anomaly score is not automatically the same thing as failure probability.**

---

## 5.3 Training Reference

The anomaly detector will primarily learn from baseline/normal observations.

Known fault labels will not be used as training targets for an unsupervised anomaly detector merely to improve its score.

Known fault labels can later be used as external evaluation evidence.

---

# 6. Why Do We Need Both?

Imagine an unfamiliar problem appears that was not represented by our known fault classes.

A classifier is designed to choose among known classes.

An anomaly detector can provide another signal:

```text
Known-class prediction
        +
High anomaly score
```

This can indicate:

> Something is unusual, even if it does not clearly match a known condition.

This makes the two components complementary.

---

# 7. Initial ML Architecture

We will keep the two ML components logically separate.

```text
                Sensor Data
                     |
              Feature Pipeline
                     |
          +----------+----------+
          |                     |
          v                     v
     Classifier          Anomaly Detector
          |                     |
          v                     v
   Fault + scores          Anomaly score
          |                     |
          +----------+----------+
                     |
                     v
             Health / Alert Logic
                     |
                     v
          Maintenance Decision Support
```

This separation makes the system easier to understand, test and improve.

---

# 8. What Are We Predicting About the Future?

Our current classifier predicts the **observed machine condition represented by the sensor behavior**.

It does not automatically predict:

> "The machine will fail in 2 hours."

For example:

```text
Sensor window
     |
     v
Model
     |
     v
faulted_bearing
```

means:

> The observed sensor pattern resembles the faulted-bearing condition represented in the training data.

It does **not** mean:

> The machine is guaranteed to fail at a particular future time.

---

# 9. RUL — Explicit Boundary

RUL means **Remaining Useful Life**.

In simple terms:

```text
Healthy
  ↓
Degradation
  ↓
More degradation
  ↓
Defined end-of-life / failure
```

RUL estimates how much useful operating life remains before that defined endpoint.

The current fault-state data does not provide a sufficiently documented run-to-failure trajectory and failure endpoint for a defensible RUL target.

Therefore:

## RUL is NOT part of the current ML problem.

We will not manufacture RUL labels from fault classes.

If a suitable run-to-failure dataset is intentionally added later, RUL can become a separate ML problem.

---

# 10. Data Leakage Constraints

Because the data is time-dependent, leakage prevention is a major requirement.

### Temporal leakage

Related neighboring observations must not make the test set unrealistically easy.

### Experiment leakage

Measurements from the same experimental run should not be split in a way that lets the model memorize experiment-specific behavior.

### Operating-condition leakage

Variables such as voltage may be associated with particular experiments.

We must check whether a feature helps because it represents machine condition or simply because it acts as a shortcut to the label.

### Preprocessing leakage

Operations such as:

- imputation;
- scaling;
- feature selection;

must be fitted using training data only.

Then they are applied to validation/test data.

---

# 11. Class Imbalance

The current System1 data contains unequal numbers of observations per class.

This is called **class imbalance**.

Class imbalance and underfitting are different:

```text
Class imbalance
→ some classes have more observations

Underfitting
→ the model cannot learn the underlying pattern sufficiently well
```

We will first measure per-class performance.

Only if imbalance materially affects the model will we consider:

- class weighting;
- carefully designed sampling;
- threshold adjustment;
- other validated approaches.

We will not blindly apply synthetic oversampling to time-dependent data.

---

# 12. Classification Evaluation

Accuracy alone will not be our only metric.

We will evaluate:

- confusion matrix;
- per-class precision;
- per-class recall;
- per-class F1-score;
- macro and weighted metrics;
- probability calibration where appropriate.

Special attention will be given to:

```text
False negatives
False positives
Minority-class performance
Temporal generalization
```

A model with high overall accuracy but poor performance on an important fault class is not automatically a good maintenance model.

---

# 13. Anomaly Detection Evaluation

The anomaly detector will be trained around baseline behavior.

Later, known fault periods can be used as external evaluation evidence:

```text
Train
  ↓
Baseline only

Test
  ↓
Baseline + known fault periods

Check:
Do known fault periods receive higher
anomaly scores?
```

This keeps anomaly training separate from fault-label evaluation.

---

# 14. Connection to Maintenance

The ML models do not directly repair the machine.

Their outputs become evidence for a health/alert layer.

Example:

```text
Predicted condition:
faulted_bearing

Anomaly:
high

Confidence:
high

        ↓

Machine-health assessment:
elevated concern

        ↓

Maintenance support:
inspection recommended
```

Exact alert thresholds and persistence rules will be determined after validation.

---

# 15. What We Are NOT Claiming

The current ML system will not claim to:

- guarantee failure prevention;
- predict an exact future failure time;
- predict RUL from the current fault-state data;
- autonomously repair or control machinery;
- treat model confidence as guaranteed physical certainty;
- claim a specific percentage reduction in factory downtime without real production evidence.

---

# 16. Final ML Definition in Simple Words

### Classification

> Give the model recent machine-condition information and ask: **"Which known condition does this behavior look like?"**

### Anomaly Detection

> Give the anomaly detector recent machine-condition information and ask: **"How different is this from normal behavior?"**

### Combined System

> Use both outputs to create machine-health information that helps maintenance personnel decide which equipment deserves investigation.

---

# 17. What Is Still Not Decided?

These decisions will be made later from evidence:

- exact feature list;
- observation-window length;
- window overlap;
- train/validation/test strategy;
- classifier algorithm;
- anomaly-detection algorithm;
- alert thresholds;
- persistence duration;
- class-imbalance strategy;
- exact role of each available dataset source.

This is intentional.

We define the problem first, then use data and experiments to choose the implementation.

---

# 18. Definition of Done

This stage is complete when we can clearly answer:

```text
What are we predicting?
        ↓
Known machine condition

What else are we detecting?
        ↓
Deviation from baseline

What is the input?
        ↓
Validated machine-condition observations

What is the classification target?
        ↓
Six known condition classes

What does one operational prediction represent?
        ↓
A recent observation window

Are we predicting exact future failure?
        ↓
No

Are we predicting RUL?
        ↓
No, not with the current data

What does the output support?
        ↓
Machine-health assessment
and maintenance decision support
```

---

# 19. Next Stage

**Part 1 → Step 4: Scope**

The Scope stage will freeze:

- V1 capabilities;
- V1 datasets;
- V1 model boundaries;
- deployment requirements;
- excluded features;
- future extensions;
- final portfolio deliverables.
