---
layout: default
title: Provider
nav_order: 5
parent: HES OP
description: "Provider mapping from HES OP hesop_clinical table"

---

# CDM Table name: PROVIDER (CDM v5.3 / v5.4)

## Reading from hesop_clinical

Use the hesop_clinical table to populate the provider table. 

![](images/image5.png)

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| provider_id | | nextval('{TARGET_SCHEMA}.sequence_pro') AS provider_id| A sequence called sequence_pro is created in the public schema to ensure unique identifier generation for provider_id’s. Firstly,the value of the sequence is determined by querying the maximum ID from a predefined source ({TARGET_SCHEMA_TO_LINK}._max_ids).The _max_ids table is established in the schema to be linked to the target schema(public), serving the purpose of storing maximum IDs for all CDM tables. This facilitates the determination of the next Provider_ID in the sequence. |
| provider_name | NULL |  |  |
| npi | NULL |  |  |
| dea |NULL  |  |  |
| specialty_concept_id | tretspef, mainspef | use tretspef OR mainspef as Speciality FROM hesop_clinical WHERE (tretspef <> '&' OR mainspef <> '&'), inorder to retrieve the target_concept_id from source_to_concept_map by joining to the source_to_concept_map with the following JOIN: LEFT JOIN source_to_concept_map as t2 on hesop_clinical.specialty = t2.source_code AND t2.source_vocabulary_id = “HES_SPEC_STCM”. ||
| care_site_id | NULL| | |
| year_of_birth | NULL |  |  |
| gender_concept_id | NULL | |  |
| provider_source_value | NULL |  | |
| specialty_source_value | tretspef, mainspef | use tretspef OR mainspef as Speciality FROM hesop_clinical WHERE (tretspef <> '&' OR mainspef <> '&'), inorder to retrieve the source_code_description from source_to_concept_map by joining to the source_to_concept_map with the following JOIN: LEFT JOIN source_to_concept_map as t2 on hesop_clinical.specialty = t2.source_code AND t2.source_vocabulary_id = “HES_SPEC_STCM”.||
| specialty_source_concept_id |NULL  |  | |
| gender_source_value | NULL| |  |
| gender_source_concept_id | NULL |  | |