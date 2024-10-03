---
layout: default
title: Observation_Period
nav_order: 3
parent: HES APC
description: "OBSERVATION_PERIOD mapping from hes_hospital table"

---


# CDM Table name: OBSERVATION_PERIOD (CDM v5.3 / v5.4)

## Reading from hes_hospital,hes_episodes.
Use the hes_hospital & hes_episodes tables to populate the observation_period table.

![](images/image3.png)

**Figure.1**

| Destination Field | Source field | Logic | Comment field |
| --- | --- | :---: | --- |
| observation_period_id |  | nextval('public.observation_period_seq') AS observation_period_id |  Autogenerate|
| person_id | patid| | |
| observation_period_start_date | admidate,epistart | Retrieve the earliest date among those dates like this: LEAST(MIN(t1.admidate), MIN(t2.epistart)) AS min_date| |
| observation_period_end_date | discharged,epiend | Retrieve the latest date among the date fields like this: GREATEST(MAX(t1.discharged), MAX(t2.epiend)) AS max_date | |
| period_type_concept_id | | 32880 | |