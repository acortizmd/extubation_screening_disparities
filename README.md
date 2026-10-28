# Language Disparities in Liberation from Invasive Mechanical Ventilation

**CLIF Version:** 2.1 OR 3.0

Measure whether patients with non-English EHR-documented language preferences move through the steps of liberation from invasive mechanical ventilation (IMV) at a different pace: SBT screening, SBT delivery after physiologic readiness, and extubation after a passed SBT.

**1st Author:** Alex Ortiz \| **Senior Author:** Gary Weissman

## Objective

Liberation from IMV depends on a sequence of steps, several of which rely on communication between clinicians, patients, and families (e.g., assessing mental status and the ability to follow commands before extubation). This study models each step as a separate time-to-event outcome to identify where along the pathway patients with a non-English language preference experience delays, if at all.

## Important: ED + Floor Data Required

> **The study cohort is adults receiving IMV, but intubation location and pre-ICU care are part of the cohort and covariate definitions.** Data from the **Emergency Department (ED)**, **procedural areas**, and **wards** are therefore needed in addition to ICU data. Sites with ICU-only data cannot participate.

ADT records must include `location_category` values for `ed`, `icu`, `procedural`, and floor locations, and `location_type` values for ICU subtypes (e.g., `medical_icu`, `general_icu`).

## Important: SBT Documentation Required

> **Steps 2 and 3 require documented spontaneous breathing trials.** Sites must map SBT documentation to `patient_assessments` with `assessment_category = "sbt_delivery_pass_fail"`. Step 1 additionally uses `sbt_screen_pass_fail` where available. The code audits how completely each site documents these items (see *Expected Results*).

## Required CLIF Tables and Fields

### 1. `patient`

| Column               | Description                        |
|----------------------|------------------------------------|
| `patient_id`         | Unique patient identifier          |
| `race_category`      | Race category                      |
| `ethnicity_category` | Ethnicity category                 |
| `sex_category`       | Sex category                       |
| `language_name`      | Language name                      |
| `language_category`  | Language category                  |
| `death_dttm`         | Date/time of death (if applicable) |

### 2. `hospitalization`

| Column                    | Description                       |
|---------------------------|-----------------------------------|
| `patient_id`              | Unique patient identifier         |
| `hospitalization_id`      | Unique hospitalization identifier |
| `admission_dttm`          | Admission date/time               |
| `discharge_dttm`          | Discharge date/time               |
| `age_at_admission`        | Age at admission (years)          |
| `discharge_category`      | Discharge disposition (incl. `hospice`, `expired`) |
| `admission_type_category` | Admission type (e.g., ED, OSH)    |

### 3. `adt`

| Column               | Description                                         |
|----------------------|-----------------------------------------------------|
| `hospitalization_id` | Unique hospitalization identifier                   |
| `hospital_id`        | Hospital identifier                                 |
| `hospital_type`      | Hospital type (e.g., academic, community)           |
| `location_category`  | Location category (`ed`, `icu`, `procedural`, floor)|
| `location_type`      | Location subtype (e.g., `medical_icu`, `general_icu`)|
| `in_dttm`            | Location entry date/time                            |
| `out_dttm`           | Location exit date/time                             |

### 4. `hospital_diagnosis`

| Column               | Description                                                            |
|----------------------|------------------------------------------------------------------------|
| `hospitalization_id` | Unique hospitalization identifier                                      |
| `diagnosis_code`     | ICD diagnosis code (used for Elixhauser Comorbidity Index calculation) |

### 5. `labs`

| Column               | Description                       |
|----------------------|-----------------------------------|
| `hospitalization_id` | Unique hospitalization identifier |
| `lab_result_dttm`    | Lab result date/time              |
| `lab_collect_dttm`   | Lab collection date/time          |
| `lab_category`       | Lab category                      |
| `lab_value_numeric`  | Numeric lab value                 |

Used for SOFA, P/F ratio, and gas-exchange features.

**Required `lab_category` values:** `creatinine`, `bilirubin_total`, `platelet_count`, `po2_arterial`, `pco2_arterial`, `ph_arterial`, `ph_venous`, `pco2_venous`, `lactate`

### 6. `vitals`

| Column               | Description                       |
|----------------------|-----------------------------------|
| `hospitalization_id` | Unique hospitalization identifier |
| `recorded_dttm`      | Vital sign recorded date/time     |
| `vital_category`     | Vital sign category               |
| `vital_value`        | Vital sign value                  |

Used for SOFA, S/F ratio, BMI, and predicted body weight.

**Required `vital_category` values:** `height_cm`, `weight_kg`, `map`, `spo2`, `heart_rate`, `respiratory_rate`, `temp_c`

### 7. `medication_admin_continuous`

| Column               | Description                       |
|----------------------|-----------------------------------|
| `hospitalization_id` | Unique hospitalization identifier |
| `admin_dttm`         | Administration date/time          |
| `med_category`       | Medication category               |
| `med_group`          | Medication group                  |
| `med_dose`           | Dose amount                       |
| `med_dose_unit`      | Dose unit                         |
| `med_route_category` | Route of medication               |

**Required `med_group` values:** `vasoactives`, `sedation`, `analgesia`, `paralytics` / `paralytic`, `pulmonary_vasodilators_iv`, `pulmonary_vasodilators_inhaled`

Vasopressor doses are combined into a norepinephrine-equivalent dose (norepinephrine, epinephrine, phenylephrine, dopamine, vasopressin). Continuous paralytic infusions (`cisatracurium`, `vecuronium`, `rocuronium`) are used in the physiologic readiness rule. Opioids are identified by `med_category`, so sites that file them under `sedation` are handled correctly.

### 8. `medication_admin_intermittent`

| Column               | Description                       |
|----------------------|-----------------------------------|
| `hospitalization_id` | Unique hospitalization identifier |
| `admin_dttm`         | Administration date/time          |
| `med_category`       | Medication category               |
| `med_group`          | Medication group                  |
| `med_dose`           | Dose amount                       |
| `med_dose_unit`      | Dose unit                         |
| `med_route_category` | Route of medication               |

**Required `med_group` values:** `paralytic` / `paralytics`, `sedation`, `analgesia`, `anxiolytic`, `cms_sepsis_qualifying_antibiotics`, `steroids`

Used for rapid-sequence intubation timing, antibiotic exposure, and steroid exposure.

### 9. `respiratory_support`

| Column                          | Description                        |
|---------------------------------|------------------------------------|
| `hospitalization_id`            | Unique hospitalization identifier  |
| `recorded_dttm`                 | Recorded date/time                 |
| `device_category`               | Respiratory device category        |
| `mode_category`                 | Ventilation mode                   |
| `tracheostomy`                  | Flag for tracheostomy presence     |
| `fio2_set`                      | FiO2 setting                       |
| `peep_set`                      | PEEP setting                       |
| `tidal_volume_set`              | Tidal volume setting               |
| `resp_rate_set`                 | Respiratory rate setting           |
| `pressure_control_set`          | Pressure control setting           |
| `pressure_support_set`          | Pressure support setting           |
| `peep_obs`                      | Measured PEEP                      |
| `plateau_pressure_obs`          | Plateau pressure                   |
| `resp_rate_obs`                 | Measured respiratory rate          |

### 10. `patient_assessments`

| Column                | Description                       |
|-----------------------|-----------------------------------|
| `hospitalization_id`  | Unique hospitalization identifier |
| `recorded_dttm`       | Recorded date/time                |
| `assessment_category` | Assessment category               |
| `numerical_value`     | Numerical assessment value        |
| `categorical_value`   | Categorical assessment value      |

**Required `assessment_category` values:** `rass`, `gcs_total`, `sbt_delivery_pass_fail`

**Used if available:** `sbt_screen_pass_fail`, `sbt_screen_performed`, `sbt_delivery_performed`, and the SBT contraindication items (`sbt_agitation`, `sbt_inadequate_oxygenation`, `sbt_intracranial_pressure`, `sbt_no_spontaneous_effort`, `sbt_vasopressure_use`)

SBT values recorded as yes/no, pass/fail, true/false, or 1/0 (in either `categorical_value` or `numerical_value`) are harmonized to 1/0. How each site records them is written to the audit files.

### 11. `microbiology_nonculture`

| Column               | Description                       |
|----------------------|-----------------------------------|
| `hospitalization_id` | Unique hospitalization identifier |
| `collect_dttm`       | Specimen collection date/time     |
| `result_dttm`        | Result date/time                  |
| `organism_category`  | Organism category                 |
| `organism_group`     | Organism group                    |
| `result_category`    | Result (e.g., `detected`)         |

Used for active respiratory viral infection flags (SARS-CoV-2, influenza, RSV, other respiratory viruses).

## Cohort Identification

The cohort includes adults receiving IMV, with each IMV run analyzed separately:

1. **Age**: 18 years or older at admission
2. **IMV runs**: Runs begin at rows where `device_category` is `"imv"` with IMV-related settings. They end after 2 or more consecutive explicit non-IMV rows OR a gap of 24 hours or more between IMV rows. Run start is corrected to rapid-sequence intubation or sedative/paralytic infusion evidence when that precedes the first ventilator charting.
3. **Tracheostomy**: Runs in patients with a prior tracheostomy are excluded; runs are ended at tracheostomy.
4. **Minimum duration**: IMV runs of at least 1 hour
5. **Study period**: Intubation from 2018 through 2025
6. **Language**: Known language preference (`language_category`); English vs non-English (Spanish and all other languages)
7. **Sex**: Known sex (male or female)

### Liberation steps

Each step is a separate time-to-event analysis with its own starting point:

| Step | Clock starts | Event |
|------|--------------|-------|
| 1 | Intubation | First documented SBT screen |
| 2 | First meeting physiologic readiness (FiO2 ≤ 0.50, PEEP ≤ 8 cm H2O, norepinephrine-equivalent ≤ 0.10 mcg/kg/min, no paralytic infusion, sustained 2 consecutive hours) | First SBT delivered (pass or fail) |
| 3 | First passed SBT | Extubation |

Death (including death or hospice discharge within 24 hours after extubation, i.e., palliative extubation), tracheostomy, and, for steps 1 and 2, extubation before the step's event are treated as competing events. Patients still on IMV at discharge or transfer are censored.

### Analysis

- **Primary**: Cause-specific Cox proportional hazards models with patient-clustered robust standard errors, stratified by hospital × ICU type, adjusted for calendar year, age, sex, Elixhauser score, intubation location, and, at the start of each step, SOFA score, oxygenation (P/F ratio, or S/F converted to a P/F equivalent), FiO2, PEEP, norepinephrine-equivalent dose, RASS, and hours on IMV.
- **Sensitivity**: unadjusted; without hospital × ICU strata; pandemic period instead of year; without RASS; + BMI; + race and ethnicity; Spanish and other non-English separately; language × period; Fine-Gray subdistribution hazards.
- **Absolute measures**: cumulative incidence at 24 hours and restricted mean time waiting over 48 hours, by language group, with patient-level bootstrap standard errors for the differences.
- **Federation**: site-level estimates are pooled at the coordinating center with random-effects meta-analysis (REML, Hartung-Knapp).

## Expected Results

Output files are written to the directory specified by `local_dir` in `extubation_screening_config.yaml`.

All files to share are in `output/FINAL_DATA_UPLOAD_ME`. **Only aggregate results are written there; counts below 11 are masked.**

**Analysis results** (`FINAL_DATA_UPLOAD_ME/project_data`):

| File | Contents |
|------|----------|
| `liberation_hr_<site>.csv` | Log hazard ratios and robust standard errors for every step × model |
| `liberation_descriptive_<site>.csv` | Events, competing events, censoring, cumulative incidence, and restricted mean time by language group |
| `liberation_absolute_<site>.csv` | Absolute differences (non-English − English) with bootstrap standard errors |
| `liberation_flow_<site>.csv` | Runs and patients at each cohort and step stage |
| `liberation_imputation_<site>.csv` | Share of each covariate imputed, by step and language group |
| `liberation_table1_<site>.csv` | Table 1 counts and summary statistics, by language group |
| `liberation_metadata_<site>.csv` | Code version, run date, and analysis parameters |

**Data quality audits** (`FINAL_DATA_UPLOAD_ME/audit`): how each site records SBT assessments, medications (continuous and intermittent), non-culture microbiology, locations, and derived features. These help identify site-specific mapping issues.

Protected intermediate files (patient-level data) are written to `intermediate_data_DO_NOT_UPLOAD` and **must not be shared**.

## Prerequisites

- **R**: version 4.0 or higher recommended
- **R packages**:

```r
packages <- c("data.table", "arrow", "fst", "lubridate", "yaml", "rprojroot",
              "survival", "splines", "sandwich", "lmtest", "parallel")
```

## Instructions

### 1. Configure `extubation_screening_config.yaml`

Rename the template file and fill in site-specific paths:

```bash
cp extubation_screening_config_template.yaml extubation_screening_config.yaml
```

```yaml
data_dir: ""       # Path to CLIF tables — must end with "/"
local_dir: ""      # Path to project directory — must end with "/"
file_type: ".parquet"
institution: "YourInstitution"
time_zone: "EST"   # Your site's local timezone
```

- `data_dir`: Path to your CLIF 2.1 parquet (or CSV) tables
- `local_dir`: Where intermediate and final results will be saved
- `file_type`: File extension of your CLIF tables (e.g. `.parquet` or `.csv`)
- `institution`: Your site name, used for labeling output files
- `time_zone`: Your site's local timezone

### 2. Run `code/00_cohort_id.Rmd` in RStudio

> **Note:** 256GB of RAM is recommended for sites with larger databases; 128GB may be sufficient for smaller databases.

### 3. Upload results to Box

Upload the contents of `FINAL_DATA_UPLOAD_ME` to the shared Box folder: [link]

## Questions

Please contact **Alex Ortiz on Slack** with any questions or issues.