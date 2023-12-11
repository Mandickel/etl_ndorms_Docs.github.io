---
layout: default
title: Person
nav_order: 1
parent: HES OP
description: "Person mapping from HES APC hes_patient table"

---

# CDM Table name: PERSON (CDM v5.3 / v5.4)

## Reading from hesop_patient

The patients mapped to the CDM from HES OP in this instance were restricted to those with a match_rank equal to one or two and the others were discarded.



![](images/image2.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| person_id | patid |  	If match_rank >= 3 discard patient (we accept only match_rank = 1 or match_rank = 2)|  If a patient has no entry in source_hesapc.hes_hospital, disregard it.Data like gender, year_of_birth, location_id, Care_site_id comes from AURUM/GOLD as the data are linked to them.|
| gender_concept_id | | | |
| year_of_birth | | | |
| month_of_birth | |  | |
| day_of_birth |  |  |  |
| birth_datetime |  |  |  |
| race_concept_id | gen_ethnicity | | race_concept_id will be mapped to SNOMED Concept_id by using gen_ethnicity to retrieve the target_concept_id from source_to_standard_vocab_map where source_vocabulary_id = "CPRD_ETHIC_STCM".|
| ethnicity_concept_id |  |  |   |
| location_id |  |  |  |
| provider_id |  |  |  |
| care_site_id | | |  |
| person_source_value | patid |  |  |
| gender_source_value | |  | |
| gender_source_concept_id |  |  |  |
| race_source_value | gen_ethnicity| | |
| race_source_concept_id |  | |
| ethnicity_source_value |  |  |  | 

## Change log

### 12-Nov-2023
- Creation of documentation