---
layout: default
title: Condition_occurrence
nav_order: 5
parent: NCRAS
description: "CONDITION_OCCURRENCE mapping from NCRAS Tumour table"

---

# CDM Table name: CONDITION_OCCURRENCE (CDM v5.3)

## Reading from Tumour to populate the CONDITION_OCCURRENCE table. 

![](images/image5.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
|condition_occurrence_id |  | |Autogenerate: if table is empty, start from MAX(public.condition_occurrence_id)+1 |
|person_id | e_patid | | |
|condition_concept_id | morph_icd10_o2, behaviour_icd10_o2, site_icd10_o2 | | CONDITION_CONCEPT_ID will be mapped to standard ICDO3 Concept_id by using morph_icd10_o2,behavior_icd10_o2,site-icd10_o2|
|condition_start_date |diagnosisdatebest  | |CONDITION_START_DATE will be mapped from diagnosisdatebest. |
|condition_start_datetime |diagnosisdatebest  |CAST(diagnosisdatebest AS DATATIME) AS CONDITION_START_DATETIME |CONDITION_START_DATETIME will be mapped from diagnosisdatebest. |
|condition_end_date |diagnosisdatebest | | CONDITION_END_DATE will be mapped from diagnosisdatebest.|
|condition_end_datetime |diagnosisdatebest  | |CONDITION_END_DATETIME will be mapped from diagnosisdatebest. |
|condition_type_concept_id |  |32818 |32818 = "EHR administration record" |
|condition_status_concept_id |  | | |
|stop_reason |  | | |
|provider_id |  | | |
|visit_occurrence_id |  | | |
|visit_detail_id |  | | |
|condition_source_value |morph_icd10_o2, behaviour_icd10_o2, site_icd10_o2 | | CONDITION_SOURCE_VALUE will be mapped to standard ICDO3 Concept_id by using morph_icd10_o2,behavior_icd10_o2,site-icd10_o2|
|condition_source_concept_id |morph_icd10_o2, behaviour_icd10_o2, site_icd10_o2 | | CONDITION_SOURCE_VALUE will be mapped to standard ICDO3 Concept_id by using morph_icd10_o2,behavior_icd10_o2,site-icd10_o2|
|condition_status_source_value |  | | |