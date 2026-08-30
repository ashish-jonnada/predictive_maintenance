# Data Strategy

## 1. Purpose

This document defines how the available predictive-maintenance data will be organized, evaluated, and used in the project.

The main rule is:

> **Raw data is preserved as source evidence. Modelling data is created later through a controlled validation and processing pipeline.**

We do not treat every available file as a separate training dataset, and we do not merge datasets simply because their columns appear similar.

---

# 2. Available Data Sources

The project currently contains two main data platforms.

## 2.1 Bently Nevada System1

This source contains controlled rotating-machinery experiments covering:

- baseline condition 1;
- baseline condition 2;
- imbalance;
- eccentric rotor;
- bent shaft;
- faulted bearing;
- faulted coupling.

The source is available in multiple representations:

- a master combined dataset;
- condition-level combined files;
- original experimental folders;
- testbed/reference data.

The different representations are retained because they provide provenance and verification value.

They are **not automatically treated as independent datasets**.

---

## 2.2 Cutsforth InsightCM

This source contains denser sensor/time-series information from the InsightCM platform.

It is available in:

- a processed tabular representation;
- raw JSON records split into multiple parts.

The raw JSON preserves the original sensor-record structure and is particularly useful for understanding ingestion and for later historical-data replay / simulated streaming.

---

# 3. Raw Data Policy

The directory:

```text
data/raw/
```

is the source-of-truth layer.

Files stored there should remain unchanged.

We do not perform the following operations directly inside `raw/`:

- deleting observations;
- renaming columns;
- changing units;
- filling missing values;
- removing outliers;
- resampling;
- scaling;
- creating model features;
- creating train/test splits.

Derived data belongs in later layers.

---

# 4. Data Layer Architecture

```text
data/
│
├── raw/
│   └── Original acquired data
│
├── interim/
│   └── Temporary validation/normalization outputs
│
├── processed/
│   └── Validated modelling datasets
│
└── features/
    └── Model-ready feature datasets
```

The flow is:

```text
Raw source
    ↓
Validation
    ↓
Interim representation
    ↓
Processing
    ↓
Validated dataset
    ↓
Feature engineering
    ↓
Model-ready features
```

---

# 5. Dataset Usage Strategy

## Bently Nevada System1

The primary role of the System1 data is **known-condition/fault classification**.

The available condition labels provide the supervised target for this task.

The original experiment structure will also be used to understand:

- experiment boundaries;
- temporal structure;
- repeated observations;
- possible leakage;
- operating-condition differences.

The condition-level files and master representation will be compared where necessary to verify consistency.

We will not count multiple representations of the same observations as additional independent training data.

---

## Cutsforth InsightCM

The InsightCM source will be evaluated for complementary condition-monitoring capabilities.

Its denser time-series structure is especially relevant to:

- time-series understanding;
- anomaly-detection experiments;
- ingestion design;
- historical replay;
- simulated real-time inference.

Its data will not be mixed with System1 data unless a specific modelling requirement and schema compatibility justify that decision.

---

# 6. Why We Do Not Blindly Merge Sources

Two datasets can contain similarly named measurements while still representing different:

- sensors;
- acquisition systems;
- sampling behavior;
- operating conditions;
- units;
- experiments;
- labeling methods.

Therefore:

```text
Similar columns
      ≠
Same data distribution
```

and:

```text
Same sensor name
      ≠
Same physical measurement
```

Any cross-source use will require explicit validation.

---

# 7. Data Quality Requirements

Before modelling, we must verify:

### Schema

- expected columns exist;
- data types are correct;
- target labels are valid.

### Time

- timestamps parse correctly;
- ordering is understood;
- sampling behavior is measured;
- repeated timestamps are investigated.

### Completeness

- missing values are quantified;
- missingness patterns are understood;
- completely missing signals are identified.

### Validity

- sensor values are physically plausible;
- units are understood;
- impossible values are investigated.

### Labels

- class names are consistent;
- label boundaries are understood;
- experiment-to-label relationships are verified.

### Independence

- repeated observations are identified;
- experiment boundaries are preserved;
- independent units of evaluation are determined.

---

# 8. Data Leakage Strategy

The project will explicitly test for:

- temporal leakage;
- experiment leakage;
- duplicate or near-duplicate observations;
- operating-condition shortcuts;
- preprocessing leakage.

Splitting will be designed only after the temporal and experimental structure is understood.

A random row-level split will not be accepted automatically.

---

# 9. Class Imbalance Strategy

Class distribution will be measured before selecting an imbalance technique.

We will compare class-level performance rather than relying only on overall accuracy.

Potential methods include:

- class weighting;
- controlled sampling;
- threshold adjustment;
- other task-appropriate approaches.

Synthetic oversampling will not be applied blindly to time-dependent sensor observations.

---

# 10. RUL Data Boundary

The current data sources are primarily condition/state experiments rather than a documented run-to-failure lifecycle with a clear end-of-life target.

Therefore:

> **RUL is not part of the current V1 modelling scope.**

We will not manufacture RUL labels from fault-condition labels.

If a suitable run-to-failure dataset is added later, it will be introduced as a separate, explicitly documented data source.

---

# 11. Training Data vs Demonstration Data

Not every source needs to become model-training data.

We distinguish:

```text
Training data
    ↓
Used to learn model parameters

Validation/test data
    ↓
Used to evaluate generalization

Reference data
    ↓
Used to understand/prove provenance

Replay data
    ↓
Used to simulate incoming sensor data

Raw source data
    ↓
Preserved as evidence
```

This prevents the project from turning every available file into a modelling input.

---

# 12. Final Data Principle

The project will use the **minimum set of validated data sources necessary to support each capability**.

The goal is not:

> "Use every file."

The goal is:

> **"Use the right data for the right engineering purpose, with traceable provenance."**

---

# 13. Data Strategy Decision

For V1:

```text
Bently Nevada System1
        ↓
Primary supervised condition/fault modelling

Cutsforth InsightCM
        ↓
Complementary time-series /
anomaly / ingestion / replay use

Original experimental files
        ↓
Provenance + validation evidence

Raw JSON
        ↓
Raw ingestion understanding +
historical replay where useful

RUL
        ↓
Excluded until suitable run-to-failure
data exists
```

This strategy is intentionally conservative. Specific feature sets and final modelling datasets will be determined after formal data validation and EDA.
