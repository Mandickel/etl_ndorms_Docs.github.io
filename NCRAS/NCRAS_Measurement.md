---
layout: default
title: Measurement
nav_order: 6
parent: NCRAS
description: "Measurement mapping from hesop_appointment tables"

---


# CDM Table name: MEASUREMENT (CDM v5.3)

## Reading from Tumour to Measurement CDM v5.3

![](images/image3.2.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
|measurement_id |  |  | Autogenerate: if table is empty, start from MAX(public.measurement_id)+1|
|person_id |e_patid  |  | |
|measurement_concept_id | histology_coded | WITH ct1 AS (SELECT histology_coded FROM Tumour) SELECT CONCEPT_ID FROM CONCEPT AS t1 INNER JOIN ct1 AS t2 ON t1.concept_code = t2.history_coded WHERE t1.domain_id = 'Measurement' AND t1.vocabulary_id = 'CPT4' AND t1.standard_concept = 'S' |MEASUREMENT_CONCEPT_ID will be mapped to standard CPT4 Concept_id by using histology_coded |
|measurement_date |diagnosisdatebest  |  | |
|measurement_datetime | diagnosisdatebest | | |
|measurement_time |  |  | |
|measurement_type_concept_id |  |32879  |32879  = "Registry" |
|operator_concept_id |  |  | |
|value_as_number |  |  | |
|value_as_concept_id | grade |  |VALUE_AS_CONCEPT_ID will be mapped to a standard SNOMED Concept_id by using NCRAS_GRADE_STCM. |
|unit_concept_id |  |  | |
|range_low |  |  | |
|range_high |  |  | |
|provider_id |  |  | |
|visit_occurrence_id |  |  | |
|visit_detail_id |  |  | |
|measurement_source_value |histology_coded |  | |
|measurement_source_concept_id |histology_coded  |  |MEASUREMENT_SOURCE_CONCEPT_ID will be mapped to standard CPT4 Concept_id by using histology_coded.  |
|unit_source_value |  |  | |
|value_source_value |histology_coded  |  | |

## Reading from Treatment to Measurement CDM v5.3

![](images/image3.png)

**Figure.2**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
|measurement_id |  |  | Autogenerate: if table is empty, start from MAX(public.measurement_id)+1|
|person_id |e_patid  |  | |
|measurement_concept_id |eventcode  |  | MEASUREMENT_CONCEPT_ID will be mapped to a standard Measurement Concept_id by using NCRAS_EVENTDESC_STCM.|
|measurement_date |eventdate  |  | |
|measurement_datetime |eventdate  |  | |
|measurement_time |  |  | |
|measurement_type_concept_id |  |32879  |32879  = "Registry" |
|operator_concept_id |  |  | |
|value_as_number | lesionsize |  | |
|value_as_concept_id |  |  | |
|unit_concept_id |  | 8588 |8588 = "millimeter" |
|range_low |  |  | |
|range_high |  |  | |
|provider_id |  |  | |
|visit_occurrence_id |  |  | |
|visit_detail_id |  |  | |
|measurement_source_value |eventcode  |  |MEASUREMENT_SOURCE_VALUE will be mapped to Standard Measurement Concept_id by using NCRAS_EVENTDESC_STCM.  |
|measurement_source_concept_id |eventcode  |  | |
|unit_source_value |  |  | |
|value_source_value |lesionsize  |  | |
