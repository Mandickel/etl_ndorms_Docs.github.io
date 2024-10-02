---
layout: default
title: UKB_HESIN to STEM
nav_order: 6
parent: UKB_HESIN
description: "Stem table description"

---

# CDM Table name: stem_table (CDM v5.4)

## Reading from hesin_diag

![](images/image9.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| id|||Removed for performance reasons|
| domain_id | NULL | | | 
| person_id | eid | | | 
| visit_occurrence_id |eid+ins_index+arr_index | | Use eid+ins_index+arr_index to retrieve visit_occurrence_id |
| visit_detail_id|eid+ins_index+arr_index ||Use eid+ins_index+arr_index to retrieve visit_detail_id |
| source_value| diag_icd9,diag_icd10 |||
| source_concept_id | diag_icd9,diag_icd10 | concept_id of either diag_icd9 or diag_icd10 | |
| type_concept_id |  | 32829 | |
| start_date | epistart | | |
| end_date | epiend | | |
| start_datetime | epistart |   | |
| concept_id  | NULL |  |  |
| end_datetime | epiend| | |
| disease_status_source_value | level| | |
| stem_source_table | | "hesin_diag" | |
 
