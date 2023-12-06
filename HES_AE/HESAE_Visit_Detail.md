---
layout: default
title: Visit_Detail
nav_order: 6
parent: HES A&E
has_children: false
description: "Visit_Detil mapping from hes_episodes tables"

---


# CDM Table name: VISIT_DETAIL (CDM v5.3 / v5.4)

## Reading from hesae_attendance.

Extract the meticulously curated data from Hospital Episode Statistics(HES) Accident & Emergency (A&E)tables, specifically, the table denoted as hesae_attendance. Subsequently, systematically transform and align this information into the corresponding OMOP CDM versions outlined below.

**Reading from hesae_attendance to Visit_Detail CDM v5.3 table:**
![](images/image4.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| visit_detail_id |  |  | Autogenerate: if table is empty, starts from MAX(public.visit_detail) + 1|
| person_id | patid |  |  |
| visit_detail_concept_id |  |  | 9201 = Inpatient visit |
| visit_detail_start_date | epistart | |  |
| visit_detail_start_datetime | epistart | |  |
| visit_detail_end_date | epiend | | |
| visit_detail_end_datetime | epiend | |  |
| visit_detail_type_concept_id |  | 32818 = "EHR administration record” |  |
| provider_id | pconsult,tretspef |  | Provider_id from PROVIDER using pconsult and tretspef |
| care_site_id | |  |  |
| visit_detail_source_value | epikey | | This will allow us to retrieve visit_detail_id using patid. |
| visit_detail_source_concept_id |  |  | 32828 |
| admitting_source_concept_id | admimeth |  | Definition to be added instead of number |
| admitting_source_value | admimeth |  | Check for OMOP codes from admimeth |
| discharge_to_concept_id | dismeth |  | Definition to be added instead of number |
| discharge_to_source_value | dismeth |  | Check for OMOP codes from dismeth |
| preceding_visit_detail_id | eorder | If eorder = 1 then 0 else check for preceding_visit_detail_id by using eorder for this patient using patid+epikey+spno. |  |
| visit_detail_parent_id |  |  |  |
| visit_occurrence_id |  |  | Use spno to retrieve visit_occurrence_id from visit_occurrence.visit_source_value |