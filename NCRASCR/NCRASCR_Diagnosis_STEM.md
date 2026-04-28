---
layout: default
title: Diagnosis
nav_order: 2
parent: NCRASCR to STEM
grand_parent: NCRASCR
description: "NCRASCR Tumour to STEM"
---

# CDM Table name: STEM
## Reading from Tumour
**ONLY NCRAS** data within the linkage_coverage period and valid in the database linked (i.e. patients do not exists in the `source_nok`) to are used in our mapping.

---
## Diagnosis

**Morph_icd10_o2, behaviour_icd10_o2, site_icd10_o2**

Cancer diagnoses are represented by concatenating the following source fields:
- `morph_icd10_o2`, 
- `behaviour_icd10_o2`
- `site_icd10_o2` 

**Format:**

![](images/cancer_diagnosis.png)

**Rule:**
- If `site_icd10_o2` has only 3 characters, append `.9`

**Example:**
8010/3-C50.9

All cancer diagnoses in the form of **[Histology]/[behaviour]-[Topography]** are ideally mapped to the **Condition** or **Measurement** domains by using the **ICDO3** vocabulary.  
When a match cannot be found in this way, site_icd10_o2 can be used on its own for the mapping using the **ICD10** vocabulary. 
When site_icd10_o2 is null then site_coded can be used for the mapping using the **ICD9CM** vocabulary. 

With this approach, all the cancer diagnoses have been mapped to Athena standard concept_ids. 
Please note that using the Athena vocabularies, a minority of diagnoses are mapped to more than one standard concept_id. These concept_ids might belong to the same domain (i.e. Condition or Measurement) or not (i.e. spread between Condition and Measurement). 

**Site_coded / site_coded_desc** 

We should use the ‘site_coded’ field ONLY when site_icd10_o2 is not populated.  

If ‘site_icd10_o2’ is not populated, the site_coded field contains 3 digits followed by “-“ or 4 digits. The coding system used seems to be ICD9CM, which report conditions in sites, and they can all be mapped. In this case, the description “site_coded_desc” is completely missing.

**basisofdiagnosis and dco**

Mappings for the “Basis of diagnosis of the tumour” (basisofdiagnosis) field, combined with the “Diagnosis death certificate only” (dco) flag, are detailed in Table S1 below: 

**Table S1: Source-to-standard mapping for the basis of tumour diagnosis field**

|basisofdiagnosis |description  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| 0 | Death certificate  | 32815  | Death Certificate  | Type Concept  | Type Concept  |
| 1 | Clinical: Diagnosis made before death without (2-7).  | 32828  | EHR episode record  | Type Concept  | Type Concept  |
| 2 | Clinical investigation: Includes all diagnostic techniques without a tissue diagnosis.  | 32828  | EHR episode record  | Type Concept  | Type Concept  |
| 4 | Specific tumour markers: Includes biochemical and/or immunological markers which are site specific. Microscopic:  | 32828  | EHR episode record  | Type Concept  | Type Concept  |
| 5 | Cytology: Examination of cells whether from a primary or secondary site, including fluids aspirated using endoscopes or needles. Also including microscopic examination of peripheral blood films and trephine bone marrow aspirates.  | 32835  | EHR Pathology report  | Type Concept  | Type Concept  |
| 6 | Histology of a metastases: Includes autopsy specimens.  | 32835  | EHR Pathology report  | Type Concept  | Type Concept  |
| 7 | Histology of a primary tumour: Includes all cutting and bone marrow biopsies. Also includes autopsy specimens of a primary tumour.  | 32835  | EHR Pathology report  | Type Concept  | Type Concept  |
| 9 | Unknown, e.g. PAS or HISS record only +  dco = 'Y'  | 32815  | Death Certificate  | Type Concept  | Type Concept  |
| 9 | Unknown, e.g. PAS or HISS record only + dco = ‘N’  | 32879  | Registry  | Type Concept  | Type Concept  |

---
## Reading from Tumour:
---
![](images/ncrascr_diag.png)


---

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| id |  | | Autogenerate |
| person_id | e_patid | | |
| visit_occurrence_id | | from visit_detail | |
| visit_detail_id | e_cr_id<br>'tumour' | Look up visit_detail_id based on the unique combination of e_cr_id and source table name | |
| concept_id | morph_icd10_o2<br>behaviour_icd10_o2<br>site_icd10_o2 | map these values in the form of [Histology]/[behavour]-[Topography] to the CDM Condition by using ICDO3 vocabulary. | |
| source_value | morph_icd10_o2<br>behaviour_icd10_o2<br>site_icd10_o2 | morph_icd10_o2 '/' behaviour_icd10_o2 '-' site_icd10_o2 '.9' | |
| source_concept_id | morph_icd10_o2<br>behaviour_icd10_o2<br>site_icd10_o2 | concept_id representing the source_value in Athna | |
| type_concept_id | basisofdiagnosis<br>dco | basisofdiagnosis is mapped by using NCRAS_TUMOUR_BASIS_DIAG_STCM.<br>if basisofdiagnosis=9, dco = 'Y' will be taken into consideration in the mapping. | |
| start_date | diagnosisdatebest | | |
| end_date | diagnosisdatebest | | |
| start_time | | '00:00:00' | |
| stem_source_table   | | 'Tumour'  | | 
| stem_source_id      | e_cr_id | | | 


### Cancer Modifiers
Cancer modifiers (e.g., stage, grade) are mapped to the Measurement table. Each modifier is linked to its corresponding diagnosis using measurement_event_id (set to the condition_occurrence_id) and meas_event_field_concept_id (set to [1147127](https://athena.ohdsi.org/search-terms/terms/1147127)). However, some cancer modifiers may not be adequately represented by Athena Measurement concepts. In cases where mapping to the Measurement domain is not feasible, these modifiers are mapped instead to the Observation table. 

**Stage:**

Stage in the form of **[stage system_]AJCC/UICC-Stage-[value]** (for example, **6th_AJCC/UICC-Stage-1A**) are mapped to the Measurement table by using the Cancer Modifier vocabulary, when stage_system is >=6. When stage_system < 6 there is no prefix before “AJCC/UICC”. [stage system] refers to stage_best_system, stage_img_system and stage_path_system, and [value] refers to stage_best, stage_img and stage_path. 

As the source data includes three sets of stage fields (_best, _img, and _path), the specific field names (e.g., stage_best, stage_img, stage_path) are stored in the value_source_value field for reference. Table 2 reports manual mapping for the Binet system present in stage_best. 

Table S3 reports manual mapping made when automatic mapping was not possible: we assigned “Tx/Nx” when T or N could not be determined. For metastasis, we assigned M1 where there was M2, M3, M4 because in the case of brain tumour, these codes could exist.  

**Table S2a: Source-to-standard mapping for cancer: stage_best**

|stage_best  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| A  | 1635315  | Binet Stage A  | Cancer Modifier  | Measurement  |
| B  | 1635179  | Binet Stage B  | Cancer Modifier  | Measurement  |
| C  | 1635062  | Binet Stage C  | Cancer Modifier  | Measurement  |
| AJCC/UICC-STAGE-2S  | 1634209  | AJCC/UICC Stage 2  | Cancer Modifier  | Measurement  |
| AJCC/UICC-STAGE-3E  | 1633650  | AJCC/UICC Stage 3  | Cancer Modifier  | Measurement  |
| AJCC/UICC-STAGE-3S  | 1633650  | AJCC/UICC Stage 3  | Cancer Modifier  | Measurement  |
| AJCC/UICC-STAGE-4S  | 1633308  | AJCC/UICC Stage 4  | Cancer Modifier  | Measurement  |
| AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| AJCC/UICC-STAGE-X  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 6TH_AJCC/UICC-STAGE-4S  | 1635323  | AJCC/UICC 6th Stage 4  | Cancer Modifier  | Measurement  |
| 6TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 6TH_AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 6TH_AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 7TH_AJCC/UICC-STAGE-2S  | 1634181  | AJCC/UICC 7th Stage 2  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-STAGE-3S  | 1633382  | AJCC/UICC 7th Stage 3  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-STAGE-4S  | 1633902  | AJCC/UICC 7th Stage 4  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 7TH_AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 7TH_AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 7TH_AJCC/UICC-STAGE-X  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 8TH_AJCC/UICC-STAGE-4S  | 1634131  | AJCC/UICC 8th Stage 4  | Cancer Modifier  | Measurement  |
| 8TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| 8TH_AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| ANN_ARBOR-1A  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-1AE  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-1AEX  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-1AX  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-1B  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-1BE  | 1633291  | Ann Arbor Stage 1  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-2A  | 1634430  | Ann Arbor Stage 2  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-2AE  | 1634430  | Ann Arbor Stage 2  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-2AX  | 1634430  | Ann Arbor Stage 2  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-2B  | 1634430  | Ann Arbor Stage 2  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-2X  | 1634430  | Ann Arbor Stage 2  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3A  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3AE  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3B  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3BE  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3BS  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3BX  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-3X  | 1635140  | Ann Arbor Stage 3  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4A  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4AE   | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4B  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4BE  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4BEX  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4BS  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4BX  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4E  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4ES  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ANN_ARBOR-4S  | 1633464  | Ann Arbor Stage 4  | Cancer Modifier  | Measurement  |
| ENETS 2007-1  | 1635838  | Stage 1  | Cancer Modifier  | Measurement  |
| ENETS 2007-1A  | 1635838  | Stage 1  | Cancer Modifier  | Measurement  |
| ENETS 2007-1B  | 1635838  | Stage 1  | Cancer Modifier  | Measurement  |
| ENETS 2007-2A  | 1635131  | Stage 2  | Cancer Modifier  | Measurement  |
| ENETS 2007-2B  | 1635131  | Stage 2  | Cancer Modifier  | Measurement  |
| ENETS 2007-3A  | 1634191  | Stage 3  | Cancer Modifier  | Measurement  |
| ENETS 2007-3B  | 1634191  | Stage 3  | Cancer Modifier  | Measurement  |
| ENETS 2007-4  | 1633987  | Stage 4  | Cancer Modifier  | Measurement  |
| ENETS 2007-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| ENETS 2007-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| FIGO-2C  | 1634127  | FIGO Stage 2  | Cancer Modifier  | Measurement  |
| FIGO-3A1I  | 1634024  | FIGO Stage 3  | Cancer Modifier  | Measurement  |
| FIGO-3A1II  | 1634024  | FIGO Stage 3  | Cancer Modifier  | Measurement  |
| FIGO-3BII  | 1634024  | FIGO Stage 3  | Cancer Modifier  | Measurement  |

**Table S2b: Source-to-standard mapping for cancer: stage_img**

|stage_img   |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| C-AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| C-AJCC/UICC-STAGE-3E  | 1634876  | AJCC/UICC clinical Stage 3  | Cancer Modifier  | Measurement  |
| C-AJCC/UICC-STAGE-3S  | 1634876  | AJCC/UICC clinical Stage 3  | Cancer Modifier  | Measurement  |
| C-AJCC/UICC-STAGE-4S  | 1634447  | AJCC/UICC clinical Stage 4  | Cancer Modifier  | Measurement  |
| C-7TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |

**Table S2c: Source-to-standard mapping for cancer: stage_path**

|stage_path    |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| P-AJCC/UICC-STAGE-2S  | 1633702  | AJCC/UICC pathological Stage 2  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-AJCC/UICC-STAGE-X  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-6TH_AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-6TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-6TH_AJCC/UICC-STAGE-X  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-7TH_AJCC/UICC-STAGE-2S  | 1634619  | AJCC/UICC 7th pathological Stage 2  | Cancer Modifier  | Measurement  |
| P-7TH_AJCC/UICC-STAGE-6  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-7TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-7TH_AJCC/UICC-STAGE-U  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-7TH_AJCC/UICC-STAGE-X  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| P-8TH_AJCC/UICC-STAGE-?  | 35919533  | Stage Unknown  | NAACCR  | Meas Value  |
| ENETS 2007-3  |  |  |  |  |


**TNM staging**

TNM staging in the form of [stage system_]AJCC/UICC-[value] (for example, 6th_AJCC/UICC-M1a) are mapped to the 
Measurement table by using Cancer Modifier vocabulary, when stage_system is >=6. When stage_system < 6 there is no 
prefix before “AJCC/UICC”. [stage system] refers to stage_best_system, stage_img_system and stage_path_system, [value] 
refers to the TNM-specific attributes, including t_best, n_best, m_best, t_img, n_img, m_img, t_path, n_path, m_path. Table S2 
reports manual mapping made when automatic mapping was not possible: see comment in the paragraph above. 

**Table S3a: Source-to-standard mapping for cancer staging: t_best / n_best / m_best**

|t_best / n_best / m_best  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| N+  | 4121194  | N+  | SNOMED  | Measurement  |
| AJCC/UICC-M1E  | 1635142  | AJCC/UICC M1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-M2  | 1635142  | AJCC/UICC M1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-M3  | 1635142  | AJCC/UICC M1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-M4  | 1635142  | AJCC/UICC M1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-M9  | 1633547  | AJCC/UICC MX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-MP  | 1633547  | AJCC/UICC MX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N00  | 1633440  | AJCC/UICC N0 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N11  | 1633885  | AJCC/UICC NX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N1P  | 1634434  | AJCC/UICC N1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N26  | 1633885  | AJCC/UICC NX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N3P  | 1635320  | AJCC/UICC N3 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-N9  | 1633885  | AJCC/UICC NX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NC0  | 1633440  | AJCC/UICC N0 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NC1  | 1634434  | AJCC/UICC N1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NCX  | 1633885  | AJCC/UICC NX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NO  | 1633440  | AJCC/UICC N0 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NP1  | 1634434  | AJCC/UICC N1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-NP2  | 1634119  | AJCC/UICC N2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T16  | 1635682  | AJCC/UICC TX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T2.  | 1635562  | AJCC/UICC T2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T2P  | 1635562  | AJCC/UICC T2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T2S  | 1635562  | AJCC/UICC T2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T3/4  | 1634654  | AJCC/UICC T4 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T30  | 1635682  | AJCC/UICC TX Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T3P  | 1634376  | AJCC/UICC T3 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-T3S  | 1634376  | AJCC/UICC T3 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TC1  | 1635564  | AJCC/UICC T1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TC2  | 1635562  | AJCC/UICC T2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TC3  | 1634376  | AJCC/UICC T3 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TC4  | 1634654  | AJCC/UICC T4 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TIS(LCIS)  | 1634530  | AJCC/UICC Tis Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TP1  | 1635564  | AJCC/UICC T1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TP2  | 1635562  | AJCC/UICC T2 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TP4  | 1634654  | AJCC/UICC T4 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TPA  | 1635114  | AJCC/UICC Ta Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TPT1  | 1635564  | AJCC/UICC T1 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TPT3  | 1634376  | AJCC/UICC T3 Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TPTA  | 1635114  | AJCC/UICC Ta Category  | Cancer Modifier  | Measurement  |
| AJCC/UICC-TS  | 1635682  | AJCC/UICC TX Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-M2  | 1633696  | AJCC/UICC 7th M1 Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-T11  | 1633390  | AJCC/UICC 7th TX Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-T2S  | 1634506  | AJCC/UICC 7th T2 Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-T3/4  | 1634576  | AJCC/UICC 7th T4 Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-T3S  | 1635854  | AJCC/UICC 7th T3 Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-TIS(LCIS)  | 1633798  | AJCC/UICC 7th Tis Category  | Cancer Modifier  | Measurement  |
| 7TH_AJCC/UICC-TS  | 1633390  | AJCC/UICC 7th TX Category  | Cancer Modifier  | Measurement  |
| 8TH_AJCC/UICC-M9  | 1634340  | AJCC/UICC 8th MX Category  | Cancer Modifier  | Measurement  |
| 8TH_AJCC/UICC-N9  | 1634216  | AJCC/UICC 8th NX Category  | Cancer Modifier  | Measurement  |
| 8TH_AJCC/UICC-T3S  | 1633528  | AJCC/UICC 8th T3 Category  | Cancer Modifier  | Measurement  |
| 8TH_AJCC/UICC-TIS(LCIS)  | 1634720  | AJCC/UICC 8th Tis Category  | Cancer Modifier  | Measurement  |

**Table S3b: Source-to-standard mapping for cancer staging: t_img / n_img / m_img**

|t_img, n_img / m_img |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| C-AJCC/UICC-T2S  | 1635033  | AJCC/UICC clinical T2 Category  | Cancer Modifier  | Measurement  |
| C-AJCC/UICC-T3S  | 1635895  | AJCC/UICC clinical T3 Category  | Cancer Modifier  | Measurement  |
| C-AJCC/UICC-M1E  | 1635085  | AJCC/UICC clinical M1 Category  | Cancer Modifier  | Measurement  |
| C-7TH_AJCC/UICC-T2S  | 1634029  | AJCC/UICC 7th clinical T2 Category  | Cancer Modifier  | Measurement  |
| C-7TH_AJCC/UICC-T3S  | 1633324  | AJCC/UICC 7th clinical T3 Category  | Cancer Modifier  | Measurement  |
| ENETS 2007-T1  | 35919386  | cT1  | NAACCR  | Meas Value  |
| ENETS 2007-T3  | 35919139  | cT3  | NAACCR  | Meas Value  |
| ENETS 2007-T4  | 35919454  | cT4  | NAACCR  | Meas Value  |
| ENETS 2007-N0  | 35919626  | cN0  | NAACCR  | Meas Value  |
| ENETS 2007-N1  | 35919638  | cN1  | NAACCR  | Meas Value  |
| ENETS 2007-M0  | 35919673  | cM0  | NAACCR  | Meas Value  |
| ENETS 2007-M1  | 35919664  | cM1  | NAACCR  | Meas Value  |
| ENETS 2007-M1B  | 35919223  | cM1b  | NAACCR  | Meas Value  |
| ENETS 2007-M1C  | 35919795  | cM1c  | NAACCR  | Meas Value  |


**Table S3c: Source-to-standard mapping for cancer staging: t_path / n_path / m_path**

|t_path, n_path / m_path |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| N+ | 35918791  | TNM Path N  | NAACCR  | Measurement  |
| P-AJCC/UICC-T2S  | 1633978  | AJCC/UICC pathological T2 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-T3S  | 1634406  | AJCC/UICC pathological T3 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-TIS(LCIS)  | 1634581  | AJCC/UICC pathological Tis Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-TS  | 1633925  | AJCC/UICC pathological TX Category  | Cancer Modifier  | Measurement  |
| P-7TH_AJCC/UICC-TIS(LCIS)  | 1634116  | AJCC/UICC 7th pathological Tis Category  | Cancer Modifier  | Measurement  |
| P-7TH_AJCC/UICC-TS  | 1635402  | AJCC/UICC 7th pathological TX Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-N9  | 1635170  | AJCC/UICC pathological NX Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-N11  | 1635170  | AJCC/UICC pathological NX Category  | Cancer Modifier  | Measurement  |
| P-8TH_AJCC/UICC-N9  | 1634117  | AJCC/UICC 8th pathological NX Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-M1E  | 1635505  | AJCC/UICC pathological M1 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-M2  | 1635505  | AJCC/UICC pathological M1 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-M3  | 1635505  | AJCC/UICC pathological M1 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-M4  | 1635505  | AJCC/UICC pathological M1 Category  | Cancer Modifier  | Measurement  |
| P-AJCC/UICC-M9  |||||
| P-7TH_AJCC/UICC-M9  |||||
| P-8TH_AJCC/UICC-M9  |||||
| ENETS 2007-T1  | 35919829  | pT1  | NAACCR  | Meas Value  |
| ENETS 2007-T1A  | 35919756  | pT1a  | NAACCR  | Meas Value  |
| ENETS 2007-T1B  | 35919186  | pT1b  | NAACCR  | Meas Value  |
| ENETS 2007-T2  | 35919074  | pT2  | NAACCR  | Meas Value  |
| ENETS 2007-T3  | 35919081  | pT3  | NAACCR  | Meas Value  |
| ENETS 2007-T4  | 35919919  | pT4  | NAACCR  | Meas Value  |
| ENETS 2007-N0  | 35919637  | pN0  | NAACCR  | Meas Value  |
| ENETS 2007-N1  | 35919098  | pN1  | NAACCR  | Meas Value  |
| ENETS 2007-N1A  | 35919353  | pN1a  | NAACCR  | Meas Value  |
| ENETS 2007-M1  | 35919480  | pM1  | NAACCR  | Meas Value  |
| ENETS 2007-M1A  | 35919199  | pM1a  | NAACCR  | Meas Value  |
| ENETS 2007-MX  | 45876323  | Mx  | LOINC  | Meas Value  |

**Grade**

Reported tumour grades and their mapping are listed in Table S4 below. We have not found a mapping in the Measurement 
domain for “G5 = T Cell”, “G6 = B Cell” and “G7 = Null cell”. 

**Table S4: Source-to-standard mapping for the grade of tumour field**

|gleason_primary |description  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| 1 | Gleason Primary Pattern Grade 1  | 1635448  | Gleason Primary Pattern Grade 1  | Cancer Modifier  | Measurement  |
| 2 | Gleason Primary Pattern Grade 2  | 1635165  | Gleason Primary Pattern Grade 2  | Cancer Modifier  | Measurement  |
| 3 | Gleason Primary Pattern Grade 3  | 1633643  | Gleason Primary Pattern Grade 3  | Cancer Modifier  | Measurement  |
| 4 | Gleason Primary Pattern Grade 4  | 1633348  | Gleason Primary Pattern Grade 4  | Cancer Modifier  | Measurement  |
| 5 | Gleason Primary Pattern Grade 5  | 1635122  | Gleason Primary Pattern Grade 5  | Cancer Modifier  | Measurement  |

**Gleason secondary**

Reported Gleason secondary pattern grades and their mapping are listed in Table S6 below.

**Table S6: Source-to-standard mapping for Gleason secondary pattern field**

|gleason_secondary |description  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| 1 | Gleason Secondary Pattern Grade 1  | 1635517  | Gleason Secondary Pattern Grade 1  | Cancer Modifier  | Measurement  |
| 2 | Gleason Secondary Pattern Grade 2  | 1635385  | Gleason Secondary Pattern Grade 2  | Cancer Modifier  | Measurement  |
| 3 | Gleason Secondary Pattern Grade 3  | 1634808  | Gleason Secondary Pattern Grade 3  | Cancer Modifier  | Measurement  |
| 4 | Gleason Secondary Pattern Grade 4  | 1633367  | Gleason Secondary Pattern Grade 4  | Cancer Modifier  | Measurement  |
| 5 | Gleason Secondary Pattern Grade 5  | 1634739  | Gleason Secondary Pattern Grade 5  | Cancer Modifier  | Measurement  |

**Gleason tertiary** 

Reported Gleason tertiary pattern grades and their mapping are listed in Table S7 below. 

**Table S7: Source-to-standard mapping for Gleason tertiary pattern field**

|gleason_tertiary |description  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| 1 | Gleason Tertiary Pattern Grade 1  | 1633991  | Gleason Tertiary Pattern Grade 1  | Cancer Modifier  | Measurement  |
| 2 | Gleason Tertiary Pattern Grade 2  | 1634870  | Gleason Tertiary Pattern Grade 2  | Cancer Modifier  | Measurement  |
| 3 | Gleason Tertiary Pattern Grade 3  | 1633295  | Gleason Tertiary Pattern Grade 3  | Cancer Modifier  | Measurement  |
| 4 | Gleason Tertiary Pattern Grade 4  | 1633951  | Gleason Tertiary Pattern Grade 4  | Cancer Modifier  | Measurement  |
| 5 | Gleason Tertiary Pattern Grade 5  | 1634454  | Gleason Tertiary Pattern Grade 5  | Cancer Modifier  | Measurement  |

**Other cancer modifiers**

Other reported cancer modifiers and their mapping are listed in Table S8 below. 

**Table S8: Source-to-standard mapping for other cancer modifiers**

|field name  |description  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| bigtumourcount | Count of every tumour associated with this e_cr_patid in range C00-C97 excluding C44  | 4078513  | Number of neoplasms in excised tissue specimen  | SNOMED  | Observation  |
| chrl_tot_78_06 | Charlson co-morbidity score over 6 years  | 42538860  | Charlson Comorbidity Index  | SNOMED  | Measurement  |
| chrl_tot_27_03 | Charlson co-morbidity score over 2 years  | 42538860  | Charlson Comorbidity Index  | SNOMED  | Measurement  |
| gleason_combined | Gleason combined  | 619648  | Gleason scoring system for malignant neoplasm of prostate  | SNOMED  | Measurement  |
| nodesexcised | The number of local and regional nodes reported as being positive  | 40479359  | Number of regional lymph nodes containing metastatic neoplasm in excised specimen  | SNOMED  | Measurement  |
| nodesinvolved | The number of local and regional nodes examined  | 4156565  | Number of regional lymph nodes examined by microscopy in excised specimen  | SNOMED  | Measurement  |
| tumourcount | Count of every tumour associated with this e_cr_patid  | 4078513  | Number of neoplasms in excised tissue specimen  | SNOMED  | Observation  |
| tumoursize | Diameter of a tumour in mm, largest if more than one  | 4154259  | Tumor size, largest dimension  | SNOMED  | Measurement  |
| npi | Nottingham Prognostic Indicator score (for prognosis following surgery for breast cancer)  | 4215959  | Nottingham prognostic index  | SNOMED  | Measurement  |
| breslow | Thickness of tumour (for staging melanoma of the skin) in mm (number of range)  | 37021218  | Breslow thickness [Length] of Skin melanoma  | LOINC  | Measurement  |

**Vitalstatus and vitalstatusdate are not mapped**

We use the death information from the ONS mortality registry ONLY. 

**route_code / final_route**

We have ignored the route_code and use the “final_route” field instead. Options are mapped as follows: 

|final_route |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| TWW = 2 week wait  | 4046885  | Urgent referral  | SNOMED  | Observation  |
| GP referral  | 44787814  | Referred by general practitioner  | SNOMED  | Observation  |
| Emergency presentation  | 4163685  | Emergency department patient visit  | SNOMED  | Observation  |
| Screening  | 44803659  | Screening  | SNOMED  | Observation  |
|  |  | Referred by hospital outpatient department  | SNOMED  | Observation  |
| Other outpatient  | 44809116  |  |  |  |
| Inpatient elective  | 4228491  | Planned admission  | SNOMED  | Observation  |
| DCO  | Already mapped by the DCO field  |  |  |  |

**er_status = Oestrogen receptor status of the tumour (for breast cancer)**

|value_source_value |source_value |value_as_concept_id  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- | --- |
| er_status  |  |  | 4307360  | Estrogen receptor assay (ERA)  | SNOMED  | Measurement  |
|  | P | 35919678  |  |  | NAACCR  | Meas Value  |
|  | N | 35919055  |  |  | NAACCR  | Meas Value  |

**er_score = Oestrogen receptor score of the tumour (for breast cancer)**

|value_source_value |source_value |value_as_concept_id  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- | --- |
| er_score ||  | 35918819  | Estrogen Receptor Total Allred Score  | NAACCR  | Measurement  |
|  | 0 | 35919095  |  |  | NAACCR  | Meas Value  |
|  | 1 | 35919804  |  |  | NAACCR  | Meas Value  |
|  | 2 | 35919902  |  |  | NAACCR  | Meas Value  |
|  | 3 | 35919181  |  |  | NAACCR  | Meas Value  |
|  | 4 | 35919373  |  |  | NAACCR  | Meas Value  |
|  | 5 | 35919036  |  |  | NAACCR  | Meas Value  |
|  | 6 | 35919497  |  |  | NAACCR  | Meas Value  |
|  | 7 | 35919196  |  |  | NAACCR  | Meas Value  |
|  | 8 | 35919763  |  |  | NAACCR  | Meas Value  |


**pr_status = Progesterone receptor status of the tumour (for breast cancer)**

|value_source_value |source_value |value_as_concept_id  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- | --- |
| pr_status |  |  | 4030384  | Progesterone receptor assay measurement  | SNOMED  | Measurement  |
|  | P | 35919362  |  |  | NAACCR  | Meas Value  |
|  | N | 35919700  |  |  | NAACCR  | Meas Value  |


**pr_score = Progesterone receptor score of the tumour (for breast cancer)**

|value_source_value |source_value |value_as_concept_id  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- | --- |
| pr_score  |  |  | 35918626  | Progesterone Receptor Total Allred Score  |NAACCR |Measurement |
|  | 0  | 35919026  |  |  |NAACCR |Meas Value |
|  | 1  | 35919439  |  |  |NAACCR |Meas Value |
|  | 2  | 35919607  |  |  |NAACCR |Meas Value |
|  | 3  | 35919054  |  |  |NAACCR |Meas Value |
|  | 4  | 35919602  |  |  |NAACCR |Meas Value |
|  | 5  | 35919915  |  |  |NAACCR |Meas Value |
|  | 6  | 35919748  |  |  |NAACCR |Meas Value |
|  | 7  | 35919716  |  |  |NAACCR |Meas Value |
|  | 8  | 35919200  |  |  |NAACCR |Meas Value |


**her2_status = Human Epidermal Growth Factor Receptor 2 status of the tumour (for breast cancer)**

|her2_status |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| P  | 44790895  | Human epidermal growth factor receptor 2 gene positive  | SNOMED  | Measurement  |
| N  | 44790896  | Human epidermal growth factor receptor 2 gene negative  | SNOMED  | Measurement  |

**dukes - Dukes stage of disease at diagnosis (for colorectal cancer)**

|dukes |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| A  | 4191720  | Dukes stage A  | SNOMED  | Condition  |
| B  | 4216304  | Dukes stage B  | SNOMED  | Condition  |
| C  | 1635190  | Duke stage C  | Cancer Modifier  | Measurement  |
| C1  | 4192885  | Dukes stage C1  | SNOMED  | Condition  |
| C2  | 4215419  | Dukes stage C2  | SNOMED  | Condition  |
| D  | 4192886  | Dukes stage D  | SNOMED  | Condition  |

**figo - Fédération Internationale de Gynécologie et d'Obstétrique - Staging system (for tumours of the female reproductive system)**

|figo  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| 1  | 1634826  | FIGO Stage 1  | Cancer Modifier  | Measurement  |
| 1A  | 1633471  | FIGO Stage 1A  | Cancer Modifier  | Measurement  |
| 1A1  | 1633850  | FIGO Stage 1A1  | Cancer Modifier  | Measurement  |
| 1A2  | 1635128  | FIGO Stage 1A2  | Cancer Modifier  | Measurement  |
| 1B  | 1633590  | FIGO Stage 1B  | Cancer Modifier  | Measurement  |
| 1B1  | 1635463  | FIGO Stage 1B1  | Cancer Modifier  | Measurement  |
| 1B2  | 1635572  | FIGO Stage 1B2  | Cancer Modifier  | Measurement  |
| 1C  | 1635358  | FIGO Stage 1C  | Cancer Modifier  | Measurement  |
| 1C1  | 1633792  | FIGO Stage 1C1  | Cancer Modifier  | Measurement  |
| 1C2  | 1634475  | FIGO Stage 1C2  | Cancer Modifier  | Measurement  |
| 1C3  | 1633311  | FIGO Stage 1C3  | Cancer Modifier  | Measurement  |
| 2  | 1634127  | FIGO Stage 2  | Cancer Modifier  | Measurement  |
| 2A  | 1633317  | FIGO Stage 2A  | Cancer Modifier  | Measurement  |
| 2A1  | 1634026  | FIGO Stage 2A1  | Cancer Modifier  | Measurement  |
| 2A2  | 1633963  | FIGO Stage 2A2  | Cancer Modifier  | Measurement  |
| 2B  | 1635822  | FIGO Stage 2B  | Cancer Modifier  | Measurement  |
| 2C (mapped as 2)  | 1634127  | FIGO Stage 2  | Cancer Modifier  | Measurement  |
| 3  | 1634024  | FIGO Stage 3  | Cancer Modifier  | Measurement  |
| 3A  | 1634412  | FIGO Stage 3A  | Cancer Modifier  | Measurement  |
| 3A1  | 1635742  | FIGO Stage 3A1  | Cancer Modifier  | Measurement  |
| 3A1I  | 1635060  | FIGO Stage 3A1(I)  | Cancer Modifier  | Measurement  |
| 3A1II  | 1633791  | FIGO Stage 3A1(II)  | Cancer Modifier  | Measurement  |
| 3A2  | 1634108  | FIGO Stage 3A2  | Cancer Modifier  | Measurement  |
| 3B  | 1635339  | FIGO Stage 3B  | Cancer Modifier  | Measurement  |
| 3BII  | No stage, no mapping  |  |  |  |
| 3C  | 1633903  | FIGO Stage 3C  | Cancer Modifier  | Measurement  |
| 3C1  | 1635141  | FIGO Stage 3C1  | Cancer Modifier  | Measurement  |
| 3C2  | 1633554  | FIGO Stage 3C2  | Cancer Modifier  | Measurement  |
| 4  | 1634306  | FIGO Stage 4  | Cancer Modifier  | Measurement  |
| 4A  | 1635588  | FIGO Stage 4A  | Cancer Modifier  | Measurement  |
| 4B  | 1635186  | FIGO Stage 4B  | Cancer Modifier  | Measurement  |

**screendetected / screeningstatuscosd_code / screeningstatuscosd_name / screeningstatusfull_code**

There is no generic concept_id for this information. There are concept_ids specific for some types of cancer (i.e. bowel, breast) 

**screeningstatusfull_name**

|screeningstatusfull_name  |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- |
| Breast: Screen Detected  | 44811954  | Breast cancer detected by national screening programme  | SNOMED  | Condition  |
| Bowel: Screen Detected  | 44791581  | Bowel cancer detected by national screening programme  | SNOMED  | Condition  |
| Bowel: Screen Detected (Extended 1 Month)  |  |  |  |  |
| Bowel: Screen Detected (Extended 2 Months)  |  |  |  |  |
| Bowel: Screen Detected (Extended 3 Months)  |  |  |  |  |
| Bowel: Surveillance Detected  |  |  |  |  |
| Cervix: Screen Detected  | 4064913  | Ca cervix screening abnormal  | SNOMED  | Condition  |
| Cervix: Screen Detected, delayed  |  |  |  |  |
| Screen detected (after routine smear resulting in suspend code but <=6mths after)  |  |  |  |  |
| Screen detected delayed (after routine smear resulting in suspend code but >6mths after)  |  |  |  |  |


**screeningstatusfull_name + site_icd10_o2**

|screeningstatusfull_name | site_icd10_o2 |target_concept_id  |target_concept_name  |target_vocabulary_id |target_domain_id |
| --- | --- | --- | --- | --- | --- |
| - Cancers detected by national screening programme
  - SC Screening Detected Cancer 
  - SC 2 - Screening Detected Cancer | C50.0
C50.1 
C50.2
C50.3
C50.4 
C50.5 
C50.6
C50.8
C50.9
D05.0
D05.1
D05.7
D05.9
D48.6 | 44811954  | Breast cancer detected by national screening programme  | SNOMED  | Condition  |
| - Cancers detected by national screening programme
  - SC Screening Detected Cancer  | C53.0
C53.9
C54.1
D06.9
  | 4064913  | Ca cervix screening abnormal | SNOMED  | Condition  |




| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| id |  | | Autogenerate |
| person_id | e_patid | | |
| visit_occurrence_id | | from visit_detail | |
| visit_detail_id | e_cr_id<br>'tumour' | Look up visit_detail_id based on the unique combination of e_cr_id and source table name | |
| concept_id | tumoursize<br>nodesexcised, nodesinvolved<br>tumourcount, bigtumourcount<br>chrl_tot_27_03, chrl_tot_78_06<br>grade<br>stage_best, stage_img, stage_path<br>t_best, n_best, m_best<br>t_img, n_img, m_img<br>t_path, n_path, m_path<br>gleason_primary, gleason_secondary, gleason_tertiary, gleason_combined | source_value is mapped by using Cancer Modifier, NCRAS_TUMOUR_MODIFIER_STCM, NCRAS_TUMOUR_GRADE_STCM, NCRAS_TUMOUR_GLEASON_PRI_STCM, NCRAS_TUMOUR_GLEASON_SEC_STCM, NCRAS_TUMOUR_GLEASON_TER_STCM and NCRAS_TUMOUR_BASIS_DIAG_STCM | |
| source_value | tumoursize<br>nodesexcised, nodesinvolved<br>tumourcount, bigtumourcount<br>chrl_tot_27_03, chrl_tot_78_06<br>grade<br>stage_best, stage_img, stage_path<br>t_best, n_best, m_best<br>t_img, n_img, m_img<br>t_path, n_path, m_path<br>gleason_primary, gleason_secondary, gleason_tertiary, gleason_combined | | |
| source_concept_id | tumoursize<br>nodesexcised, nodesinvolved<br>tumourcount, bigtumourcount<br>chrl_tot_27_03, chrl_tot_78_06<br>grade<br>stage_best, stage_img, stage_path<br>t_best, n_best, m_best<br>t_img, n_img, m_img<br>t_path, n_path, m_path<br>gleason_primary, gleason_secondary, gleason_tertiary, gleason_combined | concept_id representing the source_value in Athna for standard terminology. 0 for customised source_code. | |
| type_concept_id | basisofdiagnosis<br>dco | basisofdiagnosis is mapped by using NCRAS_TUMOUR_BASIS_DIAG_STCM.<br>if basisofdiagnosis=9, dco = 'Y' will be taken into consideration in the mapping. | |
| start_date | diagnosisdatebest | | |
| end_date | diagnosisdatebest | | |
| start_time | | '00:00:00' | |
| value_as_number     | tumoursize<br>nodesexcised, nodesinvolved<br>tumourcount, bigtumourcount<br>chrl_tot_27_03, chrl_tot_78_06<br>grade<br>stage_best_system, stage_best, stage_img, stage_path<br>stage_img_system, t_best, n_best, m_best<br>t_img, n_img, m_img<br>stage_path_system, t_path, n_path, m_path<br>gleason_primary, gleason_secondary, gleason_tertiary, gleason_combined | | | 
| unit_source_value   | | 'mm' for tumoursize<br>'month' for chrl_tot_27_03 and chrl_tot_78_06 | | 
| value_source_value  | source data field name | | There are 3 sets of AJCC/UICC code in source data indentified by prefix(e.g. best_, img_, and path_) in the source data field name  | 
| stem_source_table   | | 'Tumour-[modifier name]'  | | 
| stem_source_id      | e_cr_id | | | 

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