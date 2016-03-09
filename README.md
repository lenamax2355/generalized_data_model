# Outcomes Insights, Inc. Draft Data Model

We define a data model as a set of standard locations in which information of specific types should be stored, and the relationships among the tables.  It defines the end result of an extract, transform, and load (ETL) process for an arbitrary source (or raw) dataset.  The goals of our data model are five-fold:

1. To simplify table structures making them easier to understand and easier to create in the ETL process
1. To standardize selected data elements, but only when it makes the data easier to review and/or use without loss of information
1. To avoid the mapping/translation of one vocabulary to another
1. To capture the provenance of the original data in order to enhance the reproducibility of studies 
1. To enable a straightforward, subsequent ETL process to other data models, including OMOP and PCORnet

The focus of our data model is on the information in the source vocabulary (i.e., the vocabularies used in the original data).  This approach will allow us to utilize the substantial literature of validated algorithms based on the the source data vocabularies, enhancing transparency and reproducibility.  Therefore, within the data model, we do not separate clinical events into conditions, procedures, measurements, observations, or drug exposures.  They are all simply clinical events, with codes to identify them.  We store data using their original vocabularies (e.g., ICD-9, HCPCS, CPT, etc.) to retain the original expression of the underlying clinical information.  We use "detail" tables to capture additional information that is specific to a subset of data, but which is not always used (e.g., days supplied for drugs, quantities for procedures, etc.).

In particular, we avoid translating clinical codes from one vocabulary to another without strong evidence that such a process results in a simpler and/or clearer representation of the underlying information.  This is because these relationships are prone to continuous refinement, and they are not available for all vocabularies (e.g., procedure vocabularies).  Furthermore, the potential efficiencies from expressing queries using a common vocabulary (e.g., SNOMED) do not require that every code in the data be translated **during the ETL process**.  Queries that work across datasets with different source vocabularies can readily be done "on the fly."  The excellent vocabulary mappings provided by the Observational Health Data Science and Informatics (OHDSI) community make such a process easy to implement. 

The strength of our approach is that, if an algorithm involves a specific code, we will know exactly where to find that code regardless of the structure of the original (raw) data.  This allows us to use vocabulary tools and a broad library of existing algorithms to operate on the data and create all of the variables for our study datasets.  To this end, we have developed an open-source language, [ConceptQL](https://github.com/outcomesinsights/conceptql_spec), that enables us to create, store, share, and use algorithms that are designed to work on electronic health information.  Our project, [Jigsaw](http://www.jigsaw.io) leverages ConceptQL to define an dapply algorithms against data in our data model to build study datasets.  (And Jigsaw is designed so that algorithms also work across data **models**.)

Below is the current version of the schema for the OI Data Model.  We gratefully acknowledge the influence of the open-source OHDSI common data model [specifications](http://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm) on our thinking in creating our data model.  In addition, we acknowledge the influence of both PCORnet and i2b2 on our approach.  At the moment, all references to the concepts table refer to the OMOP version 5 vocabulary [table](http://www.ohdsi.org/web/athena/) maintained by OHDSI.

## people

- Demographic information about the patients in the data
- The column for *provider_id* is intended for situations where there is a defined primary care provider (e.g., HMO or CPRD data)

| column               | type   | description                                                                                                                     |
| -----------------    | ----   | -----------                                                                                                                     |
| id                   | serial | A unique identifier for each person                                                                                            |
| gender_concept_id    | int    | A foreign key that refers to an identifier in the concepts table for the unique gender of the person                            |
| birth_date           | date   | Date of birth (yyyy-mm-dd)                                                                                                     |
| race_concept_id      | int    | A foreign key that refers to an identifier in the concepts table for the unique race of the person                              |
| ethnicity_concept_id | int    | A foreign key that refers to an identifier in the concepts table for the ethnicity of the person                               |
| address_id           | int    | A foreign key to the place of residency for the person in the location table, where the detailed address information is stored |
| provider_id          | int    | A foreign key to the primary care provider the person is seeing in the provider table                                          |
| person_id_source_value | text    | Originial person identifier defined in the source data                                          |

## providers

- All non-facility providers (i.e., physicians, etc.) are listed

| column                      | type   | description                                                                        |
| -----------------           | ----   | -----------                                                                        |
| id                          | serial | A unique identifier for each provider                                             |
| provider_name               | text   | Provider name, if available                                                     |
| identifier                  | text   | Provider identifier                                                                |
| identifier_type             | text   | Type of identifier specified in identifier field  (UPIN, NPI, etc)                   |
| dea                         | text   | The Drug Enforcement Administration (DEA) number of the provider                  |
| specialty_concept_id        | int    | A foreign key to an identifier in the concepts table for specialty     |
| address_id                  | int    | A foreign key to the address of the location where the provider is practicing     |
| birth_date                  | int    | Date of birth (yyyy-mm-dd)                                                |
| gender_concept_id           | int    | A foreign key that refers to an identifier in the concepts table for the unique gender of the person               |


## facilities

- Unique records for all the facilities in the data

| column                      | type   | description                                                                        |
| -----------------           | ----   | -----------                                                                        |
| id                          | serial | A unique identifier for each facility                                             |
| facility_name               | text   | Facility name, if available                                                     |
| identifier                  | text   | Facility identifier                                                                |
| identifier_type             | text   | Type of identifier specified in identifier field  (UPIN,NPI,etc)                   |
| specialty_concept_id        | int    | A foreign key to an identifier in the concepts table for specialty |
| address_id                  | int    | A foreign key to the address of the location of the facility     |

## collections

- Groups provenances records
- For claims, records the claim level information (also referred to as "headers" in some databases)
- For EHR, records the visit level information
- Includes the place of service recorded with the record
- Can be linked with multiple records in the provenances table

| column                        | type   | description                                                                                 |
| -----------------             | ----   | -----------                                                                                 |
| id                            | serial | Surrogate key for record                                                                    |
| person_id                     | int    | FK to reference to person table                                                             |
| pos_concept_id                | int    | FK reference to concepts table representing the place of service associated with this record |
| start_date                    | date   | Start date of record (yyyy-mm-dd)                                                                  |
| end_date                      | date   | End date of record (yyyy-mm-dd)                                                                  |
| facility_id                   | int    | FK reference to facilities table                                                            |

## collections_providers

- Links one or more providers with a collection
- Each record represents an encounter between a person and a provider on a specific collection
- Captures the role, if any, the provider played on the collection (e.g., attending physician)

| column            | type   | description                                                                  |
| ----------------- | ----   | -----------                                                                  |
| collection_id          | int    | FK reference to collections table                                                 |
| provider_id       | int    | FK reference to providers table                                              |
| role_type_id      | text   | Roles providers can play in an encounter (currently a text field)         |

## provenances

- Holds information about where the clinical_codes and costs come from
- Groups clinical_codes typically occurring on the same day or at the same timed (e.g., a diagnosis and a procedure)
- provenance records are always linked to a collection records 

| column            | type   | description                                                                  |
| ----------------- | ----   | -----------                                                                  |
| id                | serial | Surrogate key for record                                                     |
| collection_id          | int    | FK reference to collections table                                                 |
| pos_concept_id    | int    | FK reference to concepts table representing the place of service associated with this record  |
| provider_id       | int    | FK for provider associated with this record                                           |
| type_concept_id                     | int   | FK reference to concepts table representing the type of provenance the record is (line, claim, etc.) |
| file_type                     | text   | Type of the file from which the record was pulled (currently a text field; for provenance purposes)      |

## clinical_codes

- Stores clinical codes from all types of records including procedures, diagnoses, drugs, etc.
  - ICD-9 (Proc and CM)
  - ICD-10 (Proc and CM)
  - SNOMED
  - Medcode (CPRD)
  - HCPCS/CPT
  - NDC
  - LOINC
- Ignores semantic distinctions about the type of information represented within a vocabulary because most vocabularies contain information from more than one domain
- One record generated for each individual code in the raw data

| column              | type   | description                                                                           |
| -----------------   | ----   | -----------                                                                           |
| id                  | serial | Surrogate key for record                                                              |
| provenance_id             | int    | FK reference to provenances table                                                              |
| person_id           | int    | FK reference to people table                                            |
| start_date          | date   | Start date of record (yyyy-mm-dd)                                                    |
| end_date            | date   | End date of record (yyyy-mm-dd)                                                    |
| clinical_concept_id | int    | FK reference to concepts table for the code assigned to the record   |
| quantity            | int    | Quantity, if available (e.g., procedures)                           |
| seq_num            | int    | The sequence number for the variable assigned (e.g. dx3 gets sequence number 3)                       |
| type_concept_id     | int    | Additional type information (ex: primary or admitting)                                 |

## measurement_details

- Additional information - measurements, observations, status, and specifications
- Text-based vocabularies should be mapped to LOINC, if possible (e.g., laboratory data indexed by text names for the lab results)
- Other vocabularies should be included in their original system (e.g., SEER variables)
  - This could be implemented by making variable names a vocabulary in themselves
- May need to add variables for "normal range", or consider a separate table for additional laboratory details

| column              | type   | description                                                                                                                                                                                                                                             |
| -----------------   | ----   | -----------                                                                                                                                                                                                                                             |
| id                  | serial | Surrogate key for record                                                                                                                                                                                                                                |
| clinical_code_id             | int    | FK reference to clinical_codes table to the associated clinical code                                                              |
| person_id           | int    | FK reference to people table                                                        |
| result_as_number     | float  | The observation result stored as a number, applicable to observations where the result is expressed as a numeric value    |
| result_as_string     | text   | The observation result stored as a string, applicable to observations where the result is expressed as verbatim text    |
| result_as_concept_id | int    | FK reference to concepts table for the result associated with the detail_concept_id (e.g., positive/negative, present/absent, low/high, etc.) |
| result_modifier_id | int    | FK reference to concepts table for result modifier (=, <, >, etc.) |
| unit_concept_id     | int    | FK reference to concepts table for the measurement units (e.g., mmol/L, mg/dL, etc.)        |
| normal_range_low     | float    | Lower bound of the normal reference range assigned by the laboratory      |
| normal_range_high     | float    | Upper bound of the normal reference range assigned by the laboratory      |
| normal_range_low_modifier_id | int    | FK reference to concepts table for result modifier (=, <, >, etc.) |
| normal_range_high_modifier_id | int    | FK reference to concepts table for result modifier (=, <, >, etc.) |

## drug_exposure_details

- To capture extra details about drug clinical_codes
- quantity of drug is stored in the clinical_codes field with the code

| column               | type   | description                                                                                                                            |
| -----------------    | ----   | -----------                                                                                                                            |
| id                   | serial | Surrogate key for record |
| clinical_code_id             | int    | FK reference to clinical_codes table to the associated clinical code                                                              |
| refills              | int    | The number of refills after the initial prescription; the initial prescription is not counted (i.e., values start with 0)              |
| days_supply          | int    | The number of days of supply as recorded in the original prescription or dispensing record                          |
| dose_form_concept_id | int    | FK reference to concepts table for the form of the drug (capsule, injection, etc.)       |
| dose_unit_concept_id | int    | FK reference to concepts table for the units in which the dose_value is expressed |
| dose_value           | float  | Numeric value for the dose of the drug |

## costs

- To capture costs (charges, paid amounts, and/or costs) for each provided service
- All costs are linked to a claim and could also be linked to a line, to align with the original data
- Do we need a column to indicate payer if there is more than 1 row associated with a cost record?
- Should revenue codes be in clinical_codes table?  Same with DRG and APC codes?

| column                        | type   | description                                                                                                                                                                       |
| -----------------             | ----   | -----------                                                                                                                                                                       |
| id                            | serial | A unique identifier for each COST record |
| provenance_id             | int    | FK reference to provenances table                                                              |
| currency_concept_id | int | FK reference to concepts table for the 3-letter code used to delineate international currencies (e.g., USD = US Dollar) |
| total_charged | float | The total amount charged by the provider of the good/service (e.g. hospital, physician pharmacy, dme provider) billed to a payer. This information is usually provided in claims data. |
| total_cost | float | Cost of service/device/drug incurred by provider/pharmacy. This field is more commonly derived from charge information.  |
| total_cost_type_concept_id | int | FK reference to concepts table for the provenance or the source of the cost data. Shows the provenance or the source of the total_cost data: Calculated from provider revenue, calculated from cost-to-charge ratio, reported from accounting database, etc. |
| total_paid | float | The total amount paid from all payers for the expenses of the service/device/drug. This field is calculated using the following formula: paid_by_payer + paid_by_patient + paid_by_primary. In claims data, this field is considered the calculated field the payer expects the provider to get reimbursed for the service/device/drug from the payer and from the patient, based on the payer's contractual obligations. |
| paid_by_payer | float | The amount paid by the Payer for the service/device/drug. In claims data, generally there is one field representing the total payment from the payer for the service/device/drug. However, this field could be a calculated field if the source data provides separate payment information for the ingredient cost and the dispensing fee. If the paid_ingredient_cost or paid_dispensing_fee fields are populated with nonzero values, the paid_by_payer field is calculated using the following formula: paid_ingredient_cost + paid_dispensing_fee. If there is more than one Payer in the source data, several cost records indicate that fact. The Payer reporting this reimbursement should be indicated under the payer_plan_id field. |
| paid_by_patient | float | The total amount paid by the Person as a share of the expenses. This field is most often used in claims data to report the contracted amount the patient is responsible for reimbursing the provider for said service/device/drug. This is a calculated field using the following formula: paid_patient_copay + paid_patient_coinsurance + paid_patient_deductible. If the source data has actual patient payments (e.g. the patient payment is not a derivative of the payer claim and there is verification the patient paid an amount to the provider), then the patient payment should have it's own cost record with a payer_plan_id set to 0 to indicate the the payer is actually the patient, and the actual patient payment should be noted under the total_paid field. The paid_by_patient field is only used for reporting a patient's responsibility reported on an insurance claim. |
| paid_patient_copay | float | The amount paid by the Person as a fixed contribution to the expenses. paid_patient_copay does contribute to the paid_by_patient variable. The paid_patient_copay field is only used for reporting a patient's copay amount reported on an insurance claim. |
| paid_patient_coinsurance | float | The amount paid by the Person as a joint assumption of risk. Typically, this is a percentage of the expenses defined by the Payer Plan after the Person's deductible is exceeded. paid_patient_coinsurance does contribute to the paid_by_patient variable. The paid_patient_coinsurance field is only used for reporting a patient's coinsurance amount reported on an insurance claim. |
| paid_patient_deductible | float | The amount paid by the Person that is counted toward the deductible defined by the Payer Plan. paid_patient_deductible does contribute to the paid_by_patient variable. The paid_patient_deductible field is only used for reporting a patient's deductible amount reported on an insurance claim. |
| paid_by_primary | float | The amount paid by a primary Payer through the coordination of benefits. paid_by_primary does contribute to the total_paid variable. The paid_by_primary field is only used for reporting a patient's primary insurance payment amount reported on the secondary payer insurance claim. If the source data has actual primary insurance payments (e.g. the primary insurance payment is not a derivative of the payer claim and there is verification another insurance company paid an amount to the provider), then the primary insurance payment should have it's own cost record with a payer_plan_id set to the applicable payer, and the actual primary insurance payment should be noted under the paid_by_payer field. |
| paid_ingredient_cost | float | The amount paid by the Payer to a pharmacy for the drug, excluding the amount paid for dispensing the drug. paid_ingredient_cost contributes to the paid_by_payer field if this field is populated with a nonzero value. |
| paid_dispensing_fee | float | The amount paid by the Payer to a pharmacy for dispensing a drug, excluding the amount paid for the drug ingredient. paid_dispensing_fee contributes to the paid_by_payer field if this field is populated with a nonzero value. |
| information_period_id | float | FK reference to the information_periods table  |
| amount_allowed | float | The contracted amount agreed between the payer and provider. This information is generally available in claims data. This is similar to the total_paid amount in that it shows what the payer expects the provider to be reimbursed after the payer and patient pay. This differs from the total_paid amount in that it is not a calculated field, but a field available directly in claims data. Use case: This will capture non-covered services. Noncovered services are indicated by an amount allowed and patient responsibility variables (copay, coinsurance, deductible) will be equal $0 in the source data. This means the patient is responsible for the total_charged value. The amount_allowed field is payer specific and the payer should be indicated by the payer_plan_id field. |
                                                                     
## addresses

- Used for persons, providers, and facilities

| column            | type   | description                                                                                                                    |
| ----------------- | ----   | -----------                                                                                                                    |
| id                | serial | A unique identifier for each geographic location                                                                              |
| address_1         | text   | Typically used for street address                                  |
| address_2         | text   | Typically used for additional detail such as building, suite, floor, etc. |
| city              | text   | The city field as it appears in the source data (should this be standardized?)                                                                               |
| state             | text   | The state field as it appears in the source data (should this be standardized to 2-letter states?)                                                                             |
| zip               | text   | The zip or postal code                                                                                                        |
| county            | text   | The county (should this be standardized to county code?)                                                                                                                    |

## deaths

- Capture mortality information including date and cause(s) of death
- Commonly populated from beneficiary or similar administrative data associated with the medical record
- Might need to check discharge status as part of ETL process to fill this out completely
- Use of *claim_id* and *line_id* is not necessary since deaths are in the clinical_codes table if they are specific diagnosis codes from an encounter
- Should this just be in the clinical_codes table?

| column                | type   | description                                                                                           |
| -----------------     | ----   | -----------                                                                                           |
| id                    | serial | Surrogate key for record 
| person_id             | int    | FK reference to people table                                                              |
| date                  | date   | Date of death (yyyy-mm-dd)                                                                                        |
| cause_concept_id      | int    | FK reference to concepts table for cause of death (typically ICD-9 or ICD-10 code)                                              |
| cause_type_concept_id | int    | FK reference to concepts table for the type of cause of death (e.g. primary, secondary, etc. ) |
| provider_id           | int    | FK reference to providers table                                                           |

## information_periods

- Captures periods for which information in each table is relevant
- Could include enrollment types (e.g., Part A, Part B, HMO) or just "observable" (as with up-to-standard data in CPRD)
- One row per person per enrollment type per table

| column            | type   | description                                                                                                                               |
| ----------------- | ----   | -----------                                                                                                                               |
| id                | serial | Surrogate key for record                                                                                                                  |
| person_id         | int    | FK reference to people table                                                                                                  |
| start_date        | date   | Start date of record (yyyy-mm-dd)                                                                                                                 |
| end_date          | date   | End date of record (yyyy-mm-dd)                                                                                                                 |
| information_type  | text   | String representing the type of data availability (e.g., insurance coverage, hospital data, up-to-standard date).  Could be concept type. |

## admission_details

- Captures details about admissions and emergency department encounters that don't go in the clinical_codes, provenances, or collections tables
- One row per admission
- Should handle this in the same way as "extra" information from exposures table and details table if some of their information is moved into clinical_codes
- Should we add "stay_type" to capture "observation stays" that are in the hospital but counted as outpatient facility visits?
- Should we add admit and discharge dates here? Currently we set the collections start and end date to admit and discharge for inpatient records at ETL.

| column            | type   | description                                                                                                                               |
| ----------------- | ----   | -----------                                                                                                                               |
| id                | serial | Surrogate key for record                                                                                                                  |
| person_id         | int    | FK reference to people table                                                                                                  |
| collection_id          | int    | FK reference to collections table                                                                                                                |
| admit_source      | text   | Database specific code indicating source of admission (e.g., ER visit, transfer, etc.)                                                               |
| discharge_location| text   | Database specific code indicating source of discharge (e.g., death, home, transfer, long-term care, etc.)                             |
| los | int   | Length of stay                            |

## concepts

- Adapted from OMOP concept table (could add other fields, like domain, if needed)

| column            | type   | description                                                                                                                               |
| ----------------- | ----   | -----------                                                                                                                               |
| id                | serial | Surrogate key for record (this is the concept_id)                                                                                                                 |
| vocabulary_id     | text   | Unique text-string identifier of the vocabulary (see OMOP or UMLS)                                                                                           |
| concept_code      | text   | Actual code as text string from the source vocabulary (e.g., "410.00" for ICD-9)                                                                                                                |
| concept_text      | text   | Text descriptor associated with the concept_code                                                              |

