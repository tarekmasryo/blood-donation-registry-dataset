# 🩸 Blood Donation Registry — Synthetic Donors, Prevalence & Compatibility

Synthetic blood donation operations data built for **portfolio-grade notebooks** and **decision-focused analytics**:
EDA → modeling → calibration → operating threshold → outreach policy → insights.

This repository provides a donor snapshot with eligibility/deferrals, donation history, rare blood types, country-level prevalence, and RBC transfusion compatibility (ABO/Rh).

> ⚠️ Synthetic dataset: safe for experimentation and teaching. Not clinical/medical ground truth.

---

## 📦 Dataset contents

### Files
- `data/blood_donation_registry_ml_ready.csv` — donor-level snapshot (30,000 × 27)
- `data/blood_population_distribution.csv` — country prevalence + population (39 × 12)
- `data/blood_compatibility_lookup.csv` — RBC compatibility matrix (64 × 4)
- `data/data_dictionary.csv` — column definitions, types, ranges, missing rules


---

## 🧾 Main table: donor snapshot

**File:** `blood_donation_registry_ml_ready.csv`

### Column groups
**🗺️ Identity & geography**
- `donor_id` (unique), `country_code`, `region`

**👤 Donor profile**
- `age`, `sex` (M/F), `bmi`
- `smoker` (0/1), `chronic_condition_flag` (0/1)

**✅ Eligibility & deferrals**
- `eligibility_status`: `eligible | temporary_deferral | permanent_deferral`
- `eligible_to_donate` (0/1)
- `deferral_reason`: `age_out_of_range | bmi_out_of_range | chronic_condition` (missing when eligible)

**📈 Donation behavior & history**
- `preferred_site`: `hospital | mobile_unit | community_camp`
- `donation_count_last_12m`, `lifetime_donation_count`
- `first_donation_year`, `years_since_first_donation`
- `last_donation_date`, `recency_days`
- `is_regular_donor` (0/1), `donor_age_at_first_donation`

**🩸 Blood context**
- `blood_type` (8 types), `is_rare_type` (0/1)
- `blood_type_country_prevalence` (joined from the prevalence table)

**🧮 Engineered score (optional)**
- `donation_propensity_score` (numeric baseline signal)

**🎯 Outcome columns**
- `donated_next_6m` (0/1)
- `next_6m_donation_count` (0–3)

---

## 🔗 How files connect

- `blood_donation_registry_ml_ready.csv.country_code`
  ↔ `blood_population_distribution.csv.country_code`

- `blood_donation_registry_ml_ready.csv.blood_type_country_prevalence`
  is derived from the matching country prevalence rows.

- `blood_compatibility_lookup.csv` defines RBC compatibility rules between donor/recipient blood types.

---

## 🎯 Recommended notebook directions

### 1) Donation likelihood (binary classification)
- Outcome: `donated_next_6m`
- Suggested evaluation: `ROC-AUC`, `PR-AUC`, `F1`, **calibration curve**, and threshold-based cost view.

### 2) Donation frequency (count prediction)
- Outcome: `next_6m_donation_count`
- Suggested evaluation: `MAE`, `RMSE` (+ optional Poisson/Ordinal baselines).

### 3) Decision policy (production-style)
Turn predicted probability into an outreach policy:
- choose an operating threshold given **capacity/budget**
- compare FP/FN cost tradeoffs
- validate calibration + stability across segments (country/region, rare types, eligibility)

### 4) Rare blood operations
- analyze availability of rare types (`is_rare_type`) by country prevalence
- compatibility-aware matching exploration using the lookup matrix

---

## ⚠️ Modeling notes (avoid leakage / shortcuts)

This dataset intentionally includes engineered/redundant fields to support different notebook styles.

- `donated_next_6m` is derived from `next_6m_donation_count` → **use one outcome only**.
- `eligible_to_donate` overlaps with `eligibility_status` → keep one for simpler baselines.
- `eligible_to_donate == 0` implies `donated_next_6m == 0` → for behavior modeling, consider training on `eligible_to_donate == 1`.
- `donation_propensity_score` is a strong engineered signal → use it for ranking/calibration baselines, but exclude it for “feature-only” benchmarks.

Recommended comparison:
1) train without `donation_propensity_score`
2) add it and measure uplift + calibration impact

---

## ✅ Data quality expectations

- `donor_id` is unique (no duplicates)
- no duplicate rows
- `recency_days` aligns with `as_of_date - last_donation_date`
- `country_code` values match the prevalence table
- compatibility lookup covers all 8×8 donor/recipient pairs

---

## 🚀 Quick start

```python
import pandas as pd

donors = pd.read_csv("data/blood_donation_registry_ml_ready.csv")
pop    = pd.read_csv("data/blood_population_distribution.csv")
compat = pd.read_csv("data/blood_compatibility_lookup.csv")

# Example: enrich donors with country population
donors_pop = donors.merge(pop[["country_code", "population_size"]], on="country_code", how="left")
print(donors.shape, donors_pop.shape)
```

---

## 📁 Suggested repository structure

```
.
├── data/
│ ├── blood_donation_registry_ml_ready.csv
│ ├── blood_population_distribution.csv
│ └── blood_compatibility_lookup.csv
├── docs/
│ └── data_dictionary.csv
└── README.md
```

---

## 🧪 Synthetic data notice

All records are synthetic/simulated for safe experimentation and teaching.
This dataset is not intended for clinical decision-making or real donor identification.

---

## 📝 License
CC BY 4.0

---

## ✍️ Author
Tarek Masryo
