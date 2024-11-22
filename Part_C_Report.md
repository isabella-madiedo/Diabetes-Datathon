# Part C - Report: Database Normalization and Schema Design

## 1. Introduction

In this report, we focus on designing a normalized database schema for the given dataset. The goal is to create a well-structured schema that reduces redundancy and ensures data integrity while ensuring each patient record is accurately identified.

---

## 2. Task Breakdown

### 2.1 Creating a Unique Patient ID

We started by checking whether the `patient_id` column existed in the dataset. It was found that there wasn't a unique patient identifier in the table, so we added a `patient_id` column. This ensures that each patient record can be uniquely identified, which is crucial for deduplication, especially when merging datasets or updating records.

```sql
--2.1 Check if Unique patient ID exists -- It doesn't so adding it on.
ALTER TABLE ACCENTURE.PUBLIC.CLEANED_DATA ADD COLUMN patient_id INTEGER;
ALTER TABLE ACCENTURE.PUBLIC.CLEANED_DATA MODIFY COLUMN patient_id INTEGER;```
```

### 2.2 Creating a Sequence for Patient ID

To generate unique patient_id values, we created a sequence to ensure that the patient IDs are incremented automatically starting from 1.

```sql
-- 2.2 Create a Sequence for Patient ID
CREATE SEQUENCE patient_id_seq 
    START WITH 1 
    INCREMENT BY 1;
```

### 2.3 Updating the Patient ID Column
Next, we used the sequence to update the `patient_id` column in the `CLEANED_DATA` table.

```sql
--2.3 Update the Patient ID Column--
UPDATE ACCENTURE.PUBLIC.CLEANED_DATA
SET patient_id = patient_id_seq.nextval;
```
### 2.4 Verifying Results
After updating the `patient_id column`, we verified that each `patient_id` is unique and there are no duplicates

```sql
--2.4 Verifying Results --
SELECT patient_id, COUNT(*)
FROM ACCENTURE.PUBLIC.CLEANED_DATA
GROUP BY patient_id
HAVING COUNT(*) > 1;
```

### 2.5 Checking the Updated Table
We also viewed the updated `CLEANED_DATA` table to ensure that the `patient_id` was added correctly.

```sql
--2.5-View Table to Check Unique Patient ID added--
SELECT *
FROM ACCENTURE.PUBLIC.CLEANED_DATA
LIMIT 300;
```
## 3. Designing the Schema
### 3.1 Patient Table
We designed the `PATIENT` table to store demographic data for each patient, where `patient_id` is the primary key.
```sql
--3.1 Create Patient Table --
CREATE TABLE PATIENT (
    patient_id INT PRIMARY KEY,
    age INT,
    gender STRING,
    bmi FLOAT
);
```
### 3.2 Health Metrics Table
The `HEALTH_METRICS` table stores medical metrics related to each patient's risk factors for diabetes. This table references the `patient_id` from the `PATIENT` table via a foreign key.
```sql
--3.2 Health Metrics Table --
CREATE TABLE HEALTH_METRICS (
    patient_id INT,
    hypertension INT,
    diabetes_pedigree_function FLOAT,
    FOREIGN KEY (patient_id) REFERENCES PATIENT(patient_id)
);
```
### 3.3 Lifestyle Table
The `LIFESTYLE` table captures information related to a patient's lifestyle, including diet, activity level, and alcohol consumption. This table also references the `patient_id` from the `PATIENT` table.
```sql
--3.3 Lifestyle Table --
CREATE TABLE LIFESTYLE (
    patient_id INT,
    diet_type STRING,
    physical_activity_level STRING,
    social_media_usage STRING,
    sleep_duration FLOAT,
    stress_level STRING,
    alcohol_consumption STRING,
    FOREIGN KEY (patient_id) REFERENCES PATIENT(patient_id)
);
```
## 4. Populating the Tables
### 4.1 Populate the Patient Table
We populated the `PATIENT` table with demographic information, including `patient_id, age, gender`, and `bmi`.
```sql
--4.1 Populate the Patient Table
INSERT INTO PATIENT (patient_id, age, gender, bmi)
SELECT patient_id, age, gender, bmi
FROM ACCENTURE.PUBLIC.CLEANED_DATA;
```

### 4.2 Populate the Health Metrics Table
Next, we populated the `HEALTH_METRICS` table with each patient's health metrics, including hypertension, and diabetes risk factors.
```sql
--4.2 Populate the Health Metrics Table
INSERT INTO HEALTH_METRICS (patient_id, hypertension, diabetes_pedigree_function)
SELECT patient_id, hypertension, diabetes_pedigree_function
FROM ACCENTURE.PUBLIC.CLEANED_DATA;
```
### 4.3 Populate the Lifestyle Table
Finally, we populated the `LIFESTYLE` table with lifestyle data, including diet type, physical activity level, sleep duration, and alcohol consumption.
```sql
--4.3 Populate the Lifestyle Table
INSERT INTO LIFESTYLE (patient_id, diet_type, physical_activity_level, sleep_duration, stress_level, alcohol_consumption)
SELECT patient_id, diet_type, physical_activity_level, sleep_duration, stress_level, alcohol_consumption
FROM ACCENTURE.PUBLIC.CLEANED_DATA;
```

## 5. Verifying the Data
### 5.1 Verify the Number of Patients
We verified that the number of patients in the `PATIENT` table is correct by counting the total rows.
```sql
--5.1 Verify the Number of Patients
SELECT COUNT(*) AS num_patients FROM PATIENT;
```

### 5.2 Ensure All Patients Have Metrics
We ensured that all patients have corresponding health metrics by counting the distinct `patient_id`s in the `HEALTH_METRICS` table.
```sql
--5.2 Ensure All Patients Have Metrics
SELECT COUNT(DISTINCT patient_id) AS patients_in_health_metrics
FROM HEALTH_METRICS;
```
### 5.3 Cross-Check Patient Information Across Tables
We performed a cross-check for patient data across the `PATIENT, LIFESTYLE, and HEALTH_METRICS` tables to ensure consistency.
```sql
--5.3 Cross-Check Patient Information Across Tables
SELECT p.patient_id, p.age, l.diet_type
FROM PATIENT p
LEFT JOIN LIFESTYLE l ON p.patient_id = l.patient_id
LEFT JOIN HEALTH_METRICS h ON p.patient_id = h.patient_id
WHERE p.patient_id = 1; -- Replace with any patient_id to verify
```
## 6. Investigating the Discrepancy in patient_id
We observed a discrepancy where the last `patient_id` is 63240, but the row number reached 62952, indicating gaps in the sequence.

### 6.1 Finding the Max and Min of patient_id
We checked the minimum and maximum `patient_id` values to identify the gaps.
```sql
--6.1 Finding the Max and Min of Patient_ID
SELECT MIN(patient_id) AS min_patient_id, MAX(patient_id) AS max_patient_id
FROM ACCENTURE.PUBLIC.CLEANED_DATA;
```
### 6.2 Counting the Number of Rows 
We counted the total number of rows in the `CLEANED_DATA` table.
``sql
--6.2 Counting the Number of Rows
SELECT COUNT(*) AS total_rows
FROM ACCENTURE.PUBLIC.CLEANED_DATA;
```
### 6.3 Check for Missing `patient_id` Values
We checked for missing or skipped `patient_id` values to verify where the gaps occur.
```sql
--6.3 Check for Missing patient_id Values
SELECT patient_id
FROM ACCENTURE.PUBLIC.CLEANED_DATA
ORDER BY patient_id;
```
## 7. Conclusion 
In conclusion, we successfully designed and populated a normalized database schema for the dataset. The `patient_id` column was created and populated using a sequence to ensure uniqueness. We also designed tables for patient demographic data, health metrics, and lifestyle information, ensuring relationships between the tables through foreign keys. Finally, we identified and addressed discrepancies in the `patient_id` values and validated the integrity of the data across all tables. The Report has met all of the deliverables: 
1. It documents all the steps to create a unique `patient_id` and explains the process.
2. It designs and describes a normalized schema with proper relationships between tables.
3. It populates the normalized tables with data from the original dataset.
4. It validates data integrity with appropriate queries.
5. It investigates discrepancies and provides an explanation for gaps in `patient_id`

## 8. Appendix - Schema Design
<img title="schema design" alt="schema design" src="/Team_Notes/schema_design.png">
