---
layout: default
title: CDM v5.4
nav_order: 2
parent: Visit Detail
grand_parent: HES APC
description: "Visit_detail v5.4 description"
---

# CDM Table name: Visit_detail (CDM v5.4)

## Reading from hes_episodes to Visit_Detail:
![](../images/image9.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| visit_detail_id |  |  | Autogenerate|
| person_id | patid |  |  |
| visit_detail_concept_id |  | [9201- Inpatient visit](https://athena.ohdsi.org/search-terms/terms/9201)|  |
| visit_detail_start_date | epistart,<br>admidate,<br>epiend | use the minimum of the first not null of (epistart,admidate,epiend )|   |
| visit_detail_start_datetime | epistart,<br>admidate,<br>epiend | |  |
| visit_detail_end_date | epiend,<br>discharged,<br>epistart | use the first not null of (epiend,discharged,epistart ) |   |
| visit_detail_end_datetime | epiend,<br>discharged,<br>epistart | |   |
| visit_detail_type_concept_id |  | [32818- EHR administration record](https://athena.ohdsi.org/search-terms/terms/32818)| |
| provider_id | pconsult | use pconsult to retrieve the provider_id from the provider table. | |
| care_site_id |NULL |  |  |
| visit_detail_source_value | epikey | | |
| visit_detail_source_concept_id |NULL  |  |  |
| admitted_from_concept_id | admimeth | use HESAPC_ADMIMETH_STCM | |
| admitted_from_source_value | admimeth | | |
| discharged_to_concept_id | dismeth |  use HESAPC_DISMETH_STCM | |
| discharged_to_source_value | dismeth | |  |
| preceding_visit_detail_id | | latest visit_detail_id before this one, for the patient if available |  |
| parent_visit_detail_id | NULL |  |  |
| visit_occurrence_id | patid,spno | Use patid & spno to retrieve visit_occurrence_id from visit_occurrence table. |  |

## Reading from hes_acp to Visit_Detail:
![](../images/image11.png)

**Figure.2**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| visit_detail_id |  |  nextval('public.sequence_vd') AS visit_detail_id | Autogenerate|
| person_id | patid |  |  |
| visit_detail_concept_id |  | [32037- Intensive care](https://athena.ohdsi.org/search-terms/terms/32037)|  |
| visit_detail_start_date | acpstar,<br>epistart | use the minimum of the first not null of (acpstar, epistart ) |  |
| visit_detail_start_datetime | acpstar,<br>epistart |  |   |
| visit_detail_end_date | acpend,<br>epiend | use the first not null of (acpend, epiend ) |  |
| visit_detail_end_datetime | acpend,<br>epiend | |  |
| visit_detail_type_concept_id |  | [32818- EHR administration record](https://athena.ohdsi.org/search-terms/terms/32818)|   |
| provider_id | hes_episodes.pconsult | use hes_episodes.pconsult to retrieve the provider_id from the provider table. |  |
| care_site_id |NULL |  |  |
| visit_detail_source_value | epikey | | This will allow to retrieve visit_details_id using patid If acpn = 1 then 0 else use acpn with patid+epikey and visit_detail_source_value = “Augmented care period (ACP)” to find the preceding_visit_detail_id  |
| visit_detail_source_concept_id | NULL |  |  |
| admitted_from_concept_id | acpsour | use acpsour to retrieve the target_concept_id from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_apc.acpsour AND t1.source_vocabulary_id = “HESAPC_ACPSOUR_STCM”. | |
| admitted_from_source_value | acpsour | use acpsour to retrieve the source_code_description from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_apc.acpsour AND t1.source_vocabulary_id = “HESAPC_ACPSOUR_STCM”. |  |
| discharged_to_concept_id | acpdisp | use acpdisp to retrieve the target_concept_id from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_apc.acpdisp AND t1.source_vocabulary_id = “HESAPC_ACPDISP_STCM”. |  |
| discharged_to_source_value  | acpdisp | use acpdisp to retrieve the source_code_description from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_apc.acpdisp AND t1.source_vocabulary_id = “HESAPC_ACPDISP_STCM”. |  |
| preceding_visit_detail_id | | If acpn = 1 then 0 else use acpn with patid+epikey and visit_detail_source_value = “Augmented care period (ACP)” to find the preceding_visit_detail_id |  |
| parent_visit_detail_id |  |  | Use patid + epikey where visit_detail_source_value= “Visit episode” to get the visit_detail_parent_id  |
| visit_occurrence_id |  |  | Use spno to retrieve visit_occurrence_id from visit_occurrence.visit_source_value   |



## Reading from hes_ccare to Visit_Detail CDM v5.4 table:
![](../images/image10.png)

**Figure.3**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| visit_detail_id |  |  nextval('public.sequence_vd') AS visit_detail_id | Autogenerate|
| person_id | patid |  |  |
| visit_detail_concept_id |  | 32037 = Intensive care |  |
| visit_detail_start_date | ccstartdate | | |
| visit_detail_start_datetime | ccstartdate | use ccstartdate if ccstarttime is null else ccstartdate::timestamp + ccstarttime::time as visit_detail_start_datetime. | |
| visit_detail_end_date | ccdisdate | | |
| visit_detail_end_datetime | ccdisdate | use ccdisdate if ccdistime is null else ccdisdate::timestamp + ccdistime::time as visit_detail_end_datetime.  |  |
| visit_detail_type_concept_id |  | 32818 = "EHR administration record” |  |
| provider_id | hes_episodes.pconsult |  | Use patid+epikey to get it (only if efficient and provider populated) |
| care_site_id | NULL|  |  |
| visit_detail_source_value | epikey | | This will allow to retrieve visit_details_id set to 0 at the end |
| visit_detail_source_concept_id | NULL |  |  |
| admitted_from_concept_id | ccadmisorc | use ccadmisorc to retrieve the target_concept_id from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_ccare.ccadmisorc AND t1.source_vocabulary_id = “HESAPC_ADMISORC_STCM”. |  |
| admitted_from_source_value | ccadmisorc | use ccadmisorc to retrieve the source_code_description from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_ccare.ccadmisorc AND t1.source_vocabulary_id = “HESAPC_ADMISORC_STCM”. |  |
| discharged_to_concept_id | ccdisdest | use ccdisdest to retrieve the target_concept_id from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_ccare.ccdisdest AND t1.source_vocabulary_id = “HESAPC_DISDEST_STCM”. | |
| discharged_to_source_value | ccdisdest | use ccdisdest to retrieve the source_code_description from source_to_concept_map by doing a LEFT JOIN to source_to_concept_map as t1 on t1.source_code = hes_ccare.ccdisdest AND t1.source_vocabulary_id = “HESAPC_DISDEST_STCM”. |  |
| preceding_visit_detail_id | | If eorder = 1 then 0 else use eorder with patid+epikey to find the preceding_visit_detail_id |  |
| parent_visit_detail_id |  |  | Use patid + epikey where visit_detail_source_value= “Visit episode” to get the visit_detail_parent_id  |
| visit_occurrence_id |  |  | Use spno to retrieve visit_occurrence_id from visit_occurrence.visit_source_value   |