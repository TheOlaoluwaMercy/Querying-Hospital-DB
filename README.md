# Querying-Hospital-DB
In this project, I queried an 18-table hospital database using SQL. The focus was on retrieving relevant data through the use of JOINs, subqueries.

**Database Diagram**
![Hospital (1)](https://github.com/user-attachments/assets/6c9246d4-5002-4d83-8ecf-05491dae8d0b)

These questions and queries simulate real-world data requests across different hospital operations such as admissions, medications, allergies,  and procedures.

**1. Retrieve details of patients treated in 2023**
```sql
SELECT
    CONCAT(prefix, ' ', first, ' ', last) AS Fullname, 
    CASE 
        WHEN marital = 'M' THEN 'Married'
        WHEN marital = 'S' THEN 'Single'
    END AS Marital_Status,
    race,
    ethnicity,
    CASE 
        WHEN gender = 'M' THEN 'Male'
        WHEN gender = 'F' THEN 'Female'
    END AS Gender,
    address
FROM patients
WHERE id IN (
    SELECT patient
    FROM encounters
    WHERE YEAR(start) = 2023
);
```
**2. Retrieve details of patients treated in 2023 who had allergies**
```sql
SELECT *
FROM (
    SELECT
        id,
        CONCAT(prefix, ' ', first, ' ', last) AS Fullname,
        CASE 
            WHEN marital = 'M' THEN 'Married'
            WHEN marital = 'S' THEN 'Single'
        END AS Marital_Status,
        race,
        ethnicity,
        CASE 
            WHEN gender = 'M' THEN 'Male'
            WHEN gender = 'F' THEN 'Female'
        END AS Gender,
        address
    FROM patients
    WHERE id IN (
        SELECT patient
        FROM encounters
        WHERE YEAR(start) = 2023
    )
) AS patients_2023
INNER JOIN allergies
    ON allergies.patient = patients_2023.id;
```
**3. Extract list of procedures carried out for each patient treated in 2023 with a running count per patient**
```sql
SELECT 
    CONCAT(p.prefix, ' ', p.first, ' ', p.last) AS Patient_Name,
    pr.description AS Procedure_Description,
    ROW_NUMBER() OVER (
        PARTITION BY p.id
        ORDER BY pr.start
    ) AS Procedure_Count
FROM (
    SELECT DISTINCT patient
    FROM encounters
    WHERE YEAR(start) = 2023
) AS e
INNER JOIN procedures AS pr
    ON e.patient = pr.patient
INNER JOIN patients AS p
    ON p.id = pr.patient
WHERE YEAR(pr.start) = 2023
ORDER BY Patient_Name, pr.start;
```
**4. Highlight latest immunisation(s) given to each patient**
```sql
SELECT  
    i1.patient,
    i1.description,
    i2.last_immunization_date
FROM immunizations AS i1
INNER JOIN (
    SELECT patient, MAX(date) AS last_immunization_date
    FROM immunizations 
    GROUP BY patient
) AS i2
    ON i1.patient = i2.patient AND i1.date = i2.last_immunization_date
ORDER BY i1.patient;
```
_A patient with the ID '59a7188a-2c62-2a19-e660-0dcfd7b7471f' was admitted. The following queries retrieve relevant clinical information about the patient, including medications, allergies, chronic conditions._

**5. List all medications prescribed during the patient’s most recent encounter**
```ssql
SELECT 
    description AS Most_Recent_Encounter_Medications
FROM medications
WHERE encounter = (
    SELECT TOP 1 id
    FROM encounters
    WHERE patient = '59a7188a-2c62-2a19-e660-0dcfd7b7471f'
    ORDER BY start DESC
);
```
**6. List all recorded allergies for a specific patient**
```sql
SELECT *
FROM allergies
WHERE patient = '59a7188a-2c62-2a19-e660-0dcfd7b7471f';
```
**7. List the chronic conditions (excluding findings) of this specific patient**
```sql
SELECT 
    description AS Conditions
FROM conditions
WHERE patient = '59a7188a-2c62-2a19-e660-0dcfd7b7471f'
    AND stop IS NULL
    AND description NOT LIKE '%finding%';
```

























