[README_repository.md](https://github.com/user-attachments/files/34425227/README_repository.md)
# SNOMED CT mapping audit

Analysis code for an ontology-based framework that evaluates the quality of local ICD-10-to-SNOMED CT mappings in an OMOP CDM environment.

For each local diagnosis term, the framework compares the SNOMED CT concept assigned by the institution (locally mapped) with the SNOMED CT concept reached through the OHDSI ICD-10 `Maps to` relationship (OHDSI-mapped). It then uses the SNOMED CT hierarchy and defining attributes to identify high-risk mappings for expert review.

## Framework

| Phase | Description | Implementation |
|---|---|---|
| 1 | Generation of local–OHDSI SNOMED CT concept pairs | Code in this repository |
| 2 | Classification of hierarchical relationships (type 1–4) | Code in this repository |
| 3 | Automated quality categorization (QC 1–3) | Code in this repository |
| 4 | Expert review and curation of QC 3 mappings | Manual expert review (described in the manuscript) |

**Phase 2 types**

- Type 1: exact match
- Type 2: IS-A relation (one concept is an ancestor of the other)
- Type 3: shared common ancestor
- Type 4: no relation

**Phase 3 categories**

- QC 1: likely acceptable
- QC 2: indeterminate, requiring review
- QC 3: high-risk, prioritized for curation

Type 2 pairs are categorized by the direct hierarchical distance between the 2 concepts (≤ 3 levels: QC 1; otherwise QC 2). Type 3 pairs are categorized by the distance from each concept to the nearest shared ancestor, total distance (the sum of these 2 distances), and Jaccard similarity of the `Has finding site` and `Has asso morph` attribute values. Thresholds for excessive distance are the 0.99 quantiles of the corresponding distance distributions. Type 4 pairs are assigned to QC 3.

## Requirements

- An OMOP CDM database (PostgreSQL) with the OMOP standardized vocabularies, including SNOMED CT and ICD-10
- Python 3 with `pandas`, `numpy`, and `psycopg2`

## Input data

The code uses standard OMOP CDM tables only.

| Table | Use |
|---|---|
| `condition_occurrence` | Clinical table of diagnosis records. Local diagnosis codes recorded in `condition_source_value` are used, restricted to records whose `condition_source_concept_id` belongs to the ICD10 vocabulary. |
| `source_to_concept_map` | Local mappings: `source_code` (local diagnosis code), `source_concept_id` (ICD-10 concept of the local term), `target_concept_id` (locally mapped SNOMED CT concept). Only source codes recorded in `condition_occurrence.condition_source_value` are used. |
| `concept` | Vocabulary table of all concepts and their attributes. Used for vocabulary, standard status, and concept names. |
| `concept_relationship` | Vocabulary table of direct relationships between concepts. Used for ICD-10 `Maps to` relationships and SNOMED CT attribute relationships. |
| `concept_ancestor` | Vocabulary table of hierarchical ancestor–descendant relationships with levels of separation. Used for the SNOMED CT hierarchy. |

## Usage

1. Open `261105_DQ_code_availability.ipynb`.
2. In the first code cell, set:
   - database connection details
   - `cdm_schema`: schema containing clinical tables (replaces `@cdm_schema` in the SQL queries)
   - `vocab_schema`: schema containing vocabulary tables (replaces `@vocab_schema` in the SQL queries)
3. Run all cells.

## Notes

- Institution-specific data, including the local diagnosis term list and mapping records, are not included in this repository.
- The SNOMED CT release used in the study is described in the manuscript.
