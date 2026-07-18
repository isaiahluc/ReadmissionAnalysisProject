# Data Dictionary — `Final_Diabetic_Data.csv`

Source: [UCI Machine Learning Repository — Diabetes 130-US hospitals for years 1999-2008](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008)
Final rows: 69,661 (one row per unique patient, first encounter only)
Final columns: 37

| Column | Type | Description |
|---|---|---|
| `race` | categorical | Patient race. `Unknown` used in place of original `?` placeholder. |
| `gender` | categorical | Patient gender. `Unknown` used in place of original `Unknown/Invalid`. |
| `age` | categorical | Age bracket, e.g. `[50-60)`. |
| `age_midpoint` | numeric | Midpoint of the age bracket (engineered), e.g. `[50-60)` → 55. Used for trend analysis. |
| `admission_type_id` | categorical | How the admission was initiated (Emergency, Urgent, Elective, Newborn, Trauma Center, Unknown), mapped from `IDS_mapping.csv`. |
| `discharge_disposition_id` | categorical | Where the patient went after discharge, mapped from `IDS_mapping.csv`. Expired/hospice dispositions removed pre-export (see below). |
| `admission_source_id` | categorical | How the patient entered the hospital (Referral, ER, Transfer, etc.), mapped from `IDS_mapping.csv`. |
| `time_in_hospital` | numeric | Length of stay in days (1–14). |
| `num_lab_procedures` | numeric | Number of lab tests performed during the encounter. |
| `num_procedures` | numeric | Number of non-lab procedures performed. Capped at 5 (Winsorized) to collapse the artificial spike at 6+. |
| `num_medications` | numeric | Number of distinct medications administered. Capped at the 99th percentile. |
| `number_outpatient` | numeric | Raw count of outpatient visits in the year before the encounter. |
| `number_outpatient_cat` | categorical (0/1/2) | Engineered bin: 0 = none, 1 = low (1–2), 2 = high (3+). Preferred over the raw column for modeling — see README. |
| `number_emergency` | numeric | Raw count of ER visits in the year before the encounter. |
| `number_emergency_cat` | categorical (0/1/2) | Engineered bin: 0 = none, 1 = low (1), 2 = high (2+). |
| `number_inpatient` | numeric | Raw count of inpatient stays in the year before the encounter. |
| `number_inpatient_cat` | categorical (0/1/2/3) | Engineered bin: 0 = none, 1 = one prior, 2 = 2–3 prior, 3 = 4+ prior. Strongest single predictor of 30-day readmission in this dataset — see README. |
| `diag_1`, `diag_2`, `diag_3` | categorical | Primary/secondary/tertiary ICD-9 diagnosis codes. `Unknown` used in place of `?`. |
| `number_diagnoses` | numeric | Number of diagnoses entered into the system for the encounter (1–16). |
| `metformin`, `repaglinide`, `nateglinide`, `chlorpropamide`, `glimepiride`, `glipizide`, `glyburide`, `pioglitazone`, `rosiglitazone`, `acarbose`, `insulin`, `glyburide_metformin` | categorical | Dosage change status for each medication: `No`, `Steady`, `Up`, `Down`. Near-zero-variance drug columns (e.g. `examide`, `citoglipton`, `troglitazone`) were dropped — see README. |
| `change` | categorical | Whether any diabetes medication dosage was changed during the encounter (`Ch` / `No`). |
| `diabetesMed` | categorical | Whether a diabetes medication was prescribed at all (`Yes` / `No`). |
| `readmitted` | **target** (0/1) | 1 = readmitted within 30 days, 0 = not readmitted within 30 days (includes never-readmitted and readmitted after 30 days). |

## Columns dropped during cleaning and why

| Column(s) | Reason |
|---|---|
| `encounter_id` | Unique identifier, no clinical meaning, cannot be a feature. |
| `patient_nbr` | Dropped after deduplication (kept first encounter per patient only). |
| `weight` | 96.9% missing (`?`). Imputing ~98K/101K rows would manufacture data. |
| `max_glu_serum` | 94.7% missing. Only recorded in the most acute cases — presence encodes severity, not glucose level. |
| `payer_code` | 39.6% missing. Encodes insurer type, not clinical risk; not actionable for a care team. |
| `medical_specialty` | 49.1% missing. Weakly proxied already by `diag_1`/`diag_2`/`diag_3`. |
| `A1Cresult` | 83.3% missing. Missingness correlates with severity ("was tested" more than "what the result was"). |
| `examide`, `citoglipton`, `acetohexamide`, `troglitazone`, `glimepiride-pioglitazone`, `metformin-rosiglitazone`, `metformin-pioglitazone`, `glipizide-metformin`, `tolbutamide`, `miglitol`, `tolazamide` | Near-zero variance — 99.96–100% single value across the entire dataset. Cannot split a meaningful decision boundary. |

## Rows removed during cleaning

- **Duplicate encounters** — 16,773 patients had 2–40 encounters each; only the first (lowest `encounter_id`, a reliable proxy for chronological order) is kept, to prevent the model from leaking patient-specific patterns across train/test splits.
- **Non-readmittable discharge dispositions** — rows where the patient was discharged to hospice or recorded as expired were removed entirely (`Expired`, `Expired at home. Medicaid only, hospice.`, `Expired in a medical facility. Medicaid only, hospice.`, `Hospice / home`, `Hospice / medical facility`). These patients cannot physically be readmitted within 30 days, and keeping them would artificially inflate the negative class.

Net effect: 101,766 raw rows → 69,661 final rows (37 columns, one row per patient).
