# FTCS2026_article

# AMI 65-Feature Documentation Package

This directory documents the definitions, four-view assignments, data types, missingness, preprocessing rules, and distance handling of the **65 clustering features** used in the primary analysis. It supplements feature information omitted from the main manuscript because of space limits. It contains no patient-level data, cluster assignments, or distance matrices.

## Data structure

- Study population: 4,435 patients with acute myocardial infarction (AMI).
- Primary clustering input: 65 features.
- Views: Clinical, Routine Laboratory, Cardiac Biomarker, and ECG.
- Missing-value code: `-1`; it is treated as missing during distance calculation and never as an observed numeric value.
- Primary distance: patient-pair reliability-weighted multiview Gower distance.

## Files

| File | Description |
| --- | --- |
| `feature_dictionary.csv` | A 65-row feature dictionary. Each row describes one feature included in the primary clustering distance. |
| `view_summary.csv` | Feature counts, cell-level missingness, and entire-view missingness for the four views. |
| `preprocessing_rules.md` | The implemented preprocessing and distance rules for continuous, categorical, circular, and derived features. |

Recommended reading order:

`view_summary.csv` -> `feature_dictionary.csv` -> `preprocessing_rules.md`

## Field conventions

In `feature_dictionary.csv`, `original_variable` contains an English rendering of the corresponding frozen analysis-table column name rather than the literal non-English column header. The `derived_from` field lists English renderings of upstream variables only for derived features. In `feature_dictionary.csv`, `missing_rate` uses 4,435 patients as its denominator. In `view_summary.csv`, `entire_view_missing_rate` uses 4,435 patients as its denominator; `cell_level_missing_rate` uses “4,435 patients x the number of features in the view” as its denominator; and `mean_patient_coverage` is the mean patient-level coverage of that view. All proportions are stored on a 0–1 scale.

The analysis data use `-1` as the sole missing-value code. Smoking and alcohol-use status use code 3 for “unknown” in the complete audit dataset, but code 3 is converted to `-1` in the primary clustering input and therefore does not contribute as a nominal category in distance calculations.

The source export did not include a separate laboratory unit dictionary. The dictionary reports analysis units for variables whose meanings are clear, including age, vital signs, and standard ECG measures. Laboratory units that cannot be verified from the available materials are recorded as `source_unit_not_documented` to avoid unsupported assumptions about the hospital assay system. If a laboratory data dictionary becomes available, only the `unit` column needs to be updated; the numeric processing used in the frozen analysis does not change.

## Variables excluded from clustering

The following variables are not among the 65 primary clustering features and do not enter distance calculation or cluster-number selection:

| Variable | Role |
| --- | --- |
| One-year all-cause mortality (`label`) | Post-clustering external clinical validation |
| Heart-function category | Post-clustering external clinical validation; original code 0 indicates missing and is converted to `-1`; valid categories 1–4 are interpreted nominally |
| Troponin assay indicator | Audit or sensitivity analysis |
| CK-MB assay indicator | Audit or sensitivity analysis |
| Missingness indicators | Missingness-pattern sensitivity analysis |

## Privacy

**This repository does not contain individual-level patient data or personally identifiable information.**

This directory contains no patient-level raw data, patient identifiers, cluster labels, or other information that could identify an individual.
