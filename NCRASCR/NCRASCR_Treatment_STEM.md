---
layout: default
title: Treatment 
nav_order: 2
parent: NCRASCR to STEM
grand_parent: NCRASCR
description: "NCRASCR Treatment to STEM"
---

# CDM Table name: STEM
---
## Reading from treatment

### Treatment

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| id |  | | Autogenerate |
| person_id | e_patid | | |
| visit_occurrence_id | | from visit_detail | |
| visit_detail_id | e_cr_id<br>'treatment' | Look up visit_detail_id based on the unique combination of e_cr_id and source table name | |
| concept_id | opcs4_code<br>eventdesc<br>radiodesc<br>chemo_all_drug | source_value is mapped by using OPCS4, RXNorm, RxNorm Extension and NCRAS_STCM. | |
| source_value | opcs4_code<br>eventdesc<br>radiodesc<br>chemo_all_drug | | |
| source_concept_id | opcs4_code<br>eventdesc<br>radiodesc<br>chemo_all_drug | concept_id representing the source_value in Athna for standard terminology. 0 for customised source_code. | |
| type_concept_id | basisofdiagnosis<br>dco | basisofdiagnosis is mapped by using NCRAS_TUMOUR_BASIS_DIAG_STCM.<br>if basisofdiagnosis=9, dco = 'Y' will be taken into consideration in the mapping. | |
| start_date | eventdate | | |
| end_date | eventdate | | |
| start_time | | '00:00:00' | |
| stem_source_table   | | 'Treatment'  | | 
| stem_source_id      | treatment_id | | | 

### Additional information about treatment

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| id |  | | Autogenerate |
| person_id | e_patid | | |
| visit_occurrence_id | | from visit_detail | |
| visit_detail_id | e_cr_id<br>'tumour' | Look up visit_detail_id based on the unique combination of e_cr_id and source table name | |
| concept_id | number_of_tumours<br>within_six_months_flag<br>six_months_after_flag<br>lesionsize | source_value is mapped by using NCRAS_TREATMENT_MODIFIER_STCM | |
| source_value | number_of_tumours | | |
| source_concept_id | number_of_tumours | 0 | |
| type_concept_id | basisofdiagnosis<br>dco | basisofdiagnosis is mapped by using NCRAS_TUMOUR_BASIS_DIAG_STCM.<br>if basisofdiagnosis=9, dco = 'Y' will be taken into consideration in the mapping. | |
| start_date | diagnosisdatebest | | |
| end_date | diagnosisdatebest | | |
| start_time | | '00:00:00' | |
| value_as_number     | number_of_tumours<br>lesionsize<br>within_six_months_flag<br>six_months_after_flag |6 for within_six_months_flag and six_months_after_flag | | 
| value_as_string     | within_six_months_flag<br>six_months_after_flag | 'Y'/'N' | |
| unit_source_value   | | 'mm' for lesionsize<br>'month' for within_six_months_flag and six_months_after_flag | | 
| qualifier_concept_id | within_six_months_flag<br>six_months_after_flag |  [4172704](https://athena.ohdsi.org/search-terms/terms/4172704) for six_months_after_flag<br>[4171754](https://athena.ohdsi.org/search-terms/terms/4171754) for within_six_months_flag | | 
| qualifier_source_value | within_six_months_flag<br>six_months_after_flag | '>' for six_months_after_flag<br>'<=' for within_six_months_flag | | 
| stem_source_table   | | 'Treatment-Modifier'  | | 
| stem_source_id      | treatment_id | | | 