# Outcomes Insights, Inc. Draft Common Data Model

The purpose of this CDM is to avoid any unnecessary mapping, translation, and restructuring of raw data.  While we are inspired by CDMs such as OMOP, we find their CDM to require a very complicated ETL process in order to get data into their desired format.

We reject the need for mapping various terminologies into a standard vocabulary.  That step is not useful for the type of research we plan on conducting in the near term.

We don't believe there is much benefit in assign a "domain" to each concept in every terminology.  Again, we aim to search through our data using ICD-9/HCPCS/CPT codes and simply need to know when, and in what context, those codes were reported.  It is not important for us to determine if a HCPCS code really represents a measurement or observation or drug exposure.  It is more important for us to know that if an algorithm involves a HCPCS code, that we know which table to look for that code.

## clinical_codes
- Instead of having separate condition and procedure tables, we’ll have all codes from the following vocabularies end up in this table.
    - Vocabularies
        - ICD-9 (Proc and CM)
        - ICD-10 (Proc and CM)
        - SNOMED
        - MEDCODE
        - HCPCS/CPT
        - SEER/Oncology
- The OMOP specification for procedure_occurrence and condition_occurrence are quite similar.  Having two separate tables follows with OMOP’s philosophy of classifying each concept into a specific domain.  Since we are not interested in domains for our research, and since the tables are so similar, there is no philosophical or technical reason why we can’t combine conditions and procedures into the same table.  Indeed, CPRD data does not make this distinction.
- For each code we find in the source data, we will create a new row in this table.  The code from the source data will be matched against OMOP’s concept table and we will save the concept_id in this table, rather than the raw code.

| column              | type   | description                                                                           |
| -----------------   | ----   | -----------                                                                           |
| id                  | serial | Surrogate key for record                                                              |
| person_id           | int    | ID of person associated with this record                                              |
| start_date          | date   | Date of when clinical record began                                                    |
| stop_date           | date   | Date of when clinical record ended                                                    |
| encounter_id        | int    | FK for encounter associated with this record                                          |
| provenance_id       | int    | FK for provenance record associated with this procedure                               |
| clinical_concept_id | int    | FK reference into concept table representing the clinical code assigned to the record |
| quantity            | int    | Sometimes quantity is reported in claims data for procedures                          |

## encounters
- Represents an encounter between a patient and one provider in a particular place of service
- Can be pointed to by multiple clinical, detail, and exposure records
- Vocabularies
    - Place of service

| column            | type   | description                                                                                 |
| ----------------- | ----   | -----------                                                                                 |
| id                | serial | Surrogate key for record                                                                    |
| person_id         | int    | ID of person associated with this record                                                    |
| start_date        | date   | Date of when record began                                                                   |
| stop_date         | date   | Date of when record ended                                                                   |
| provenance_id     | int    | FK reference to provenance table                                                            |
| provider_id       | int    | FK reference to provider table                                                              |
| visit_id          | int    | FK reference to visit table                                                                 |
| pos_concept_id    | int    | FK reference to concept table representing the place of service associated with this record |

## details
- Additional information – measurements, observations, status, and specifications
- Might eventually map to LOINC where possible

| column              | type    | description                                                                                                                                                                                                                                             |
| -----------------   | ----    | -----------                                                                                                                                                                                                                                             |
| id                  | serial  | Surrogate key for record                                                                                                                                                                                                                                |
| person_id           | int     | ID of person associated with this record                                                                                                                                                                                                                |
| start_date          | date    | Date of when record began                                                                                                                                                                                                                               |
| stop_date           | date    | Date of when record ended                                                                                                                                                                                                                               |
| encounter_id        | int     | FK reference to encounter table                                                                                                                                                                                                                         |
| provenance_id       | int     | FK reference to provenance table                                                                                                                                                                                                                        |
| detail_concept_id   | int     | FK reference to concept table representing the topic the detail addresses                                                                                                                                                                               |
| value_as_number     | float   | The observation result stored as a number. This is applicable to observations where the result is expressed as a numeric value.                                                                                                                         |
| value_as_string     | text    | The observation result stored as a string. This is applicable to observations where the result is expressed as verbatim text.                                                                                                                           |
| value_as_concept_id | Integer | A foreign key to an observation result stored as a Concept ID. This is applicable to observations where the result can be expressed as a Standard Concept from the Standardized Vocabularies (e.g., positive/negative, present/absent, low/high, etc.). |
| unit_concept_id     | integer | A foreign key to a Standard Concept ID of measurement units in the Standardized Vocabularies.                                                                                                                                                           |

## Exposures
- To capture drug/device data (outside of procedure codes).  See OMOP drug/device exposure tables
- Could include devices if they are reported separately from procedures
- Vocabularies
    - NDC
    - RxNorm
    - Prodcodes (CPRD)
    - device vocab (?)


| column               | type   | description                                                                                                                            |
| -----------------    | ----   | -----------                                                                                                                            |
| id                   | serial | Surrogate key for record                                                                                                               |
| person_id            | int    | ID of person associated with this record                                                                                               |
| start_date           | date   | Date of when record began                                                                                                              |
| stop_date            | date   | Date of when record ended                                                                                                              |
| encounter_id         | int    | FK reference to encounter table                                                                                                        |
| provenance_id        | int    | FK reference to provenance table                                                                                                       |
| provider_id          | int    | FK reference to provider table                                                                                                         |
| exposure_concept_id  | int    | FK reference to concept table representing the exposure represented by this record                                                     |
| stop_reason          | text   | The reason the Drug was stopped. Reasons include regimen completed, changed, removed, etc.                                             |
| refills              | int    | The number of refills after the initial prescription. The initial prescription is not counted, values start with 0.                    |
| quantity             | float  | The quantity of drug as recorded in the original prescription or dispensing record.                                                    |
| days_supply          | int    | The number of days of supply of the medication as recorded in the original prescription or dispensing record.                          |
| dose_unit_concept_id | int    | A foreign key to a predefined concept in the Standardized Vocabularies reflecting the unit the effective_drug_dose value is expressed. |

## deaths
- Capture mortality information – date and cause(s) of death
- Might want to check diagnosis codes as part of ETL (?)
- Possibly roll into information period table?

| column                | type   | description                                                                                           |
| -----------------     | ----   | -----------                                                                                           |
| id                    | serial | Surrogate key for record                                                                              |
| person_id             | int    | ID of person associated with this record                                                              |
| date                  | date   | Date of death                                                                                         |
| encounter_id          | int    | FK reference to encounter table                                                                       |
| provenance_id         | int    | FK reference to provenance table                                                                      |
| cause_concept_id      | int    | FK reference into concept that represents cause of death                                              |
| cause_type_concept_id | int    | FK reference into concept that represents the type of cause of death (e.g. primary, secondary, etc. ) |

## costs
- To capture costs (charges, reimbursed amounts, and/or costs) for each provided service
- OI cost table

| column                        | type  | description                                                                                                                                                                       |
| -----------------             | ----  | -----------                                                                                                                                                                       |
| id                            | serial   | A unique identifier for each COST record.                                                                                                                                         |
| cost_event_id                 | int   | A foreign key identifier to the event (e.g. Measurement, Procedure, Visit, Drug Exposure, etc) record for which cost data are recorded.                                           |
| table_name                    | text  | The name of the table where the associated event record is found.                                                                                                                 |
| currency_concept_id           | int   | A concept representing the 3-letter code used to delineate international currencies, such as USD for US Dollar.                                                                   |
| charge                        | float | The amount charged by the provider of the good/service (e.g. hospital, physician pharmacy, dme provider)                                                                          |
| paid_copay                    | float | The amount paid by the Person as a fixed contribution to the expenses. Copay does not contribute to the out of pocket expenses.                                                   |
| paid_coinsurance              | float | The amount paid by the Person as a joint assumption of risk. Typically, this is a percentage of the expenses defined by the Payer Plan after the Person's deductible is exceeded. |
| paid_toward_deductible        | float | The amount paid by the Person that is counted toward the deductible defined by the Payer Plan.                                                                                    |
| paid_by_payer                 | float | The amount paid by the Payer. If there is more than one Payer, several COST records indicate that fact.                                                                           |
| paid_by_coordination_benefits | float | The amount paid by a secondary Payer through the coordination of benefits.                                                                                                        |
| total_out_of_pocket           | float | The total amount paid by the Person as a share of the expenses.                                                                                                                   |
| total_paid                    | float | The total amount paid for the expenses of drug exposure.                                                                                                                          |
| ingredient_cost               | float | The portion of the drug expenses due to the cost charged by the manufacturer for the drug, typically a percentage of the Average Wholesale Price.                                 |
| dispensing_fee                | float | The portion of the drug expenses due to the dispensing fee charged by the pharmacy, typically a fixed amount.                                                                     |
| cost                          | float | Cost of service/device/drug incurred by provider/pharmacy.  Was "average_wholesale_price" which represented: "List price of a Drug set by the manufacturer."                      |
| information_period_id         | int   | A foreign key to the information_period table, where the details of the Payer, Plan and Family are stored.                                                                         |
| amount_allowed                | float | The contracted amount the provider has agreed to accept as payment in full.                                                                                                       |
| revenue_code_concept_id       | int   | A foreign key referring to a Standard Concept ID in the Standardized Vocabularies for Revenue codes.                                                                              |
| revenue_code_source_value     | text  | The source code for the Revenue code as it appears in the source data, stored here for reference.                                                                                 |


## people
- See OMOP person table

| column               | type | description                                                                                                                     |
| -----------------    | ---- | -----------                                                                                                                     |
| id                   | serial  | A unique identifier for each person.                                                                                            |
| gender_concept_id    | int  | A foreign key that refers to an identifier in the CONCEPT table for the unique gender of the person.                            |
| birth_date           | date | Date of birth                                                                                                                   |
| race_concept_id      | int  | A foreign key that refers to an identifier in the CONCEPT table for the unique race of the person.                              |
| ethnicity_concept_id | int  | A foreign key that refers to the standard concept identifier in the Standardized Vocabularies for the ethnicity of the person.  |
| location_id          | int  | A foreign key to the place of residency for the person in the location table, where the detailed address information is stored. |
| provider_id          | int  | A foreign key to the primary care provider the person is seeing in the provider table.                                          |
| care_site_id         | int  | A foreign key to the site of primary care in the care_site table, where the details of the care site are stored.                |

## locations
- See OMOP location table – used for persons and care sites

| column            | type | description                                                                                                                    |
| ----------------- | ---- | -----------                                                                                                                    |
| id                | serial  | A unique identifier for each geographic location.                                                                              |
| address_1         | text | The address field 1, typically used for the street address, as it appears in the source data.                                  |
| address_2         | text | The address field 2, typically used for additional detail such as buildings, suites, floors, as it appears in the source data. |
| city              | text | The city field as it appears in the source data.                                                                               |
| state             | text | The state field as it appears in the source data.                                                                              |
| zip               | text | The zip or postal code.                                                                                                        |
| county            | text | The county.                                                                                                                    |


## providers
- See OMOP provider table.  Adapt to allow multiple providers via encounter table

| column                      | type | description                                                                        |
| -----------------           | ---- | -----------                                                                        |
| id                          | serial  | A unique identifier for each Provider.                                             |
| provider_name               | text | A description of the Provider.                                                     |
| npi                         | text | The National Provider Identifier (NPI) of the provider.                            |
| dea                         | text | The Drug Enforcement Administration (DEA) number of the provider.                  |
| specialty_concept_id        | int  | A foreign key to a Standard Specialty Concept ID in the Standardized Vocabularies. |
| care_site_id                | int  | A foreign key to the main Care Site where the provider is practicing.              |
| birth_date                  | int  | The date of birth of the Provider.                                                 |
| gender_concept_id           | int  | The gender of the Provider.                                                        |
| specialty_source_concept_id | int  | A foreign key to a Concept that refers to the code used in the source.             |
| gender_source_concept_id    | int  | A foreign key to a Concept that refers to the code used in the source.             |


## care_sites
- See OMOP care site table.

| column                        | type | description                                                                                                                        |
| -----------------             | ---- | -----------                                                                                                                        |
| id                            | serial  | A unique identifier for each Care Site.                                                                                            |
| care_site_name                | text | The description or name of the Care Site                                                                                           |
| place_of_service_concept_id   | int  | A foreign key that refers to a Place of Service Concept ID in the Standardized Vocabularies.                                       |
| location_id                   | int  | A foreign key to the geographic Location of the Care Site in the LOCATION table, where the detailed address information is stored. |
| care_site_source_value        | text | The identifier for the Care Site in the source data, stored here for reference.                                                    |
| place_of_service_source_value | text | The source code for the Place of Service as it appears in the source data, stored here for reference.                              |


## information_periods
- Captures periods for which information in each table is relevant.  Could include enrollment types (e.g., Part A, Part B, HMO) or just “observable” (as with up-to-standard data in CPRD)
- One row per person per enrollment type per table

| column            | type   | description                                                                                                                                                           |
| ----------------- | ----   | -----------                                                                                                                                                           |
| id                | serial | Surrogate key for record                                                                                                                                              |
| person_id         | int    | ID of person associated with this record                                                                                                                              |
| start_date        | date   | Date of when record began                                                                                                                                             |
| stop_date         | date   | Date of when record ended                                                                                                                                             |
| provenance_id     | int    | FK reference to provenance table                                                                                                                                      |
| enrollment_type   | text   | String representing the type of insurance                                                                                                                             |
| applicable_table  | text   | name of the table which this period provides information for.  E.g. Part D enrollment implies data for the exposure table (though not for device exposures...hmmmmmm) |

## provenances
- Records information about where a row in the CDM came from
- Most tables will have a provenance_id pointing to a row in this table
- If we split some of the information in this table into another table, we won’t need to make a new row for EVERY row in the other tables, we just need to make a row for each unique combination of the values for the columns below, i.e. clinical rows may share a common provenance_id

| column                 | type   | description                                                                                                                                                 |
| -----------------      | ----   | -----------                                                                                                                                                 |
| id                     | serial | Surrogate key for record                                                                                                                                    |
| file_name              | text   | Name of the file from which the record was pulled                                                                                                           |
| file_year              | int    | Year in which the file was ??????                                                                                                                           |
| file_row_id            | text   | ID assigned to the original row from which the record was pulled                                                                                            |
| position               | int    | The position for the variable assigned e.g. dx3 gets position 3                                                                                             |
| original_variable_name | text   | Name of the original variable from which the record was derived.  This won’t work for details since more than one field might contribute to a detail record |

## Miscellaneous details and questions
- Do we need a table for “facility”, “hospitalization” or “extended care” records (with types for inpatient, long-term, SNF, etc.)
- What about modifiers – tend to be for laterality (left/right) or multiple physicians and maybe part of ETL
- Do we need some types in the data (e.g., “cancer registry”, “claims”, “EHR”)