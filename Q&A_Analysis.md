# Medical Data Analysis
## Questions and Answers

**Author**: Chen Chen <br />
**Email**: cc4865@nyu.edu <br />
**Website**: chencapp.github.io <br />
**LinkedIn**: www.linkedin.com/in/cchengwnyu <br />

**1.**  List the number of total cases admitted between 2015 to 2017 

````sql
SELECT 
	to_char(count(*), '9g999g999') as "Total Cases" 
FROM 
	medical.conditions;
````

**Results:**

Total Cases|
---------------------|
1,252        |

**2.** List the top 10 conditions with the most admitted cases after 2015.

````sql
CREATE TABLE top_cond AS
SELECT 
	description AS "Condition",
  	count(description) as "Number of Cases" 
FROM 
	med.conditions
WHERE 
	start>= '2015-01-01' 
GROUP BY 
	"Condition"
ORDER BY 
	"Number of Cases" DESC
LIMIT
  	10;

SELECT * FROM top_cond
````

**Results:**

Condition|Number of Cases|
----------|--------|
Viral sinusitis (disorder)|  302|
Acute Viral pharyngitis (disorder)  |  169|
Acute Bronchitis (disorder) |    145|
Normal Pregnancy  | 77|
Otitis media | 49|
Sprain of Ankle | 36|
Streptococcal Sore Throat (disorder)|33|
Concussion with no loss of consciousness| 18|
Laceration of forearm | 14|
Facial laceration | 14|

**3.** What are the top 10 conditoins with the highest prevalence rate? How much overlap is there with answer to Question 2? <br>

**Conditions with Highest Prevalence Rate:**
````sql
CREATE TABLE top_prev AS
SELECT
	item as "Condition", "PREVALENCE RATE" AS "Prevalence rate"
FROM
	all_prevalences
ORDER BY
	"PREVALENCE RATE" DESC
LIMIT 10;

SELECT * FROM top_prev
````
Condition|Prevalence rate|
----------|--------|
Viral sinusitis (disorder)|  0.868|
Acute Viral pharyngitis (Disorder)  |  0.772|
Acute Bronchitis (Disorder) |    0.749|
Otitis media | 0.699|
Streptococcal Sore Throat (disorder)|0.487|
Sprain of Ankle| 0.367|
Normal Pregnancy| 0.34|
Hypertension | 0.238|
Prediabetes|0.232|

**Overlap with Question 2**
````sql
SELECT COUNT(*) as "Number of Duplicates" FROM
(SELECT Condition FROM top_cond
 INTERSECT 
 SELECT Condition FROM top_prev;) Dupes
````

**Number of Duplicates**|
-----|
7|

**4.** List number of cases and year-on-year growth of sinusitis cases by year of the most recent 10 years.
````sql
SELECT 
	SUBSTRING(START, 1, 4) AS year,
  	count(DESCRIPTION) as "Number of Sinusitis",
    	FORMAT((count(DESCRIPTION)/LAG(count(DESCRIPTION)) OVER (ORDER BY SUBSTRING(START, 1, 4))-1), 'P1')  AS "YOY % Change"
FROM 
	conditions
WHERE 
	description like '%sinusitis%'
GROUP BY 
	year
ORDER BY 
	year ASC
LIMIT
  	10;
````

Year|Number of Sinusitis| YoY % CHANGE|
----------|--------|--|
2008|  112|   NULL 
2009 |  167|  49.1%|
2010 |    124| -25.7% |
2011  | 110| -11.3%|
2012| 127| 15.5%|
2013| 128| 0.7%|
2014|99|-22.7%|
2015| 133|34.3%|
2016 | 117|-12.0%|
2017 | 83|-29.1%|



**5.** List the top 5 patients with the most amount of diagnosis after 2010.

```sql
SELECT 
	patient,
	count(description) AS "Number of Diagnosis"
FROM 
	med.conditions
WHERE
	start>='2010-01-01'
GROUP BY
	patient
ORDER BY 
	"Number of Diagnosis" DESC
LIMIT
	5;
````

**Results:**
PATIENT   |Number of Diagnosis|
---------------|--------|
d9e0ea5d-ebab-45a4-8bf2-bd4298da9f7d          | 14|           
68f094bb-dd13-4a96-844a-9c509effd8de    |  12|           
a09bba57-86de-46a7-9a24-1147547921f6|  12|           
0bcc9845-c873-492e-96e1-9771ebcbc2df|10|
2297617f-c6ce-4f63-9445-72527391a02d|10|

**6.** For each year, what percentage of patients diagnosed with Viral sinusitis (disorder) are vaccinated with the influenza vaccine? 
````sql
SELECT
	SUBSTRING(immunizations.DATE, 1, 4) AS vaccination_year,
	COUNT(case when conditions.DESCRIPTION LIKE "%sinusitis%" then 1 else 0 end) as Sinusitis_Patients,
	COUNT(case when conditions.DESCRIPTION LIKE "%sinusitis%" AND immunizations.DESCRIPTION LIKE "%Influenza%" then 1 else 0 end) as Vaccinated_Sinusitis_Patients,
	COUNT(case when conditions.DESCRIPTION LIKE "%sinusitis%" then 1 else 0 end) / COUNT(case when conditions.DESCRIPTION LIKE "%sinusitis%" AND immunizations.DESCRIPTION LIKE "%Influenza%" then 1 else 0 end) AS "% Vaccinated"
FROM
	conditions
INNER JOIN
	immunizations
ON
	conditions.PATIENT = immunizations.PATIENT
Where 
	conditions.DESCRIPTION LIKE "%sinusitis%"
GROUP BY
	vaccination_year
ORDER BY
	vaccination_year
````

**Results:**

Year      |Sinusitis_Patients    |Vaccinated_Sinusitis_Patients    |% Vaccinated|
---------------|----------|---|--|
2007           |     249  |238  | 95.6%
2008           |     1764 |1612 | 91.4% 
2009 	       |     1737 |1688 | 97.2%
2010           |     1795 |1703 | 94.9%
2011           |     1563 |1456 | 93.2%
2012           |     1568 |1486 | 94.77%
2013   	       |     1514 |1495 | 98.7%
2014 	       |     1511 |1477 | 97.7%
2015           |     1359 |1289 | 94.8%
2016           |     1380 |1238 | 89.7%
2017	       |     1102 |1006 | 91.3%

**7.** What are the top 5 conditions that occured the most prescriptions after 2015? What are the most commonly prescribed drugs for each?
````sql
WITH TopDiagnoses AS (
    SELECT
	Diagnosis,
	COUNT(*) AS "Total Number of Prescriptions"
    FROM
        medications
    WHERE
        start >= '2015-01-01'
        AND Diagnosis <> ''
    GROUP BY
        Diagnosis
    ORDER BY
        "Total Number of Prescriptions" DESC
    LIMIT 5
),
TotalPrescriptions AS (
    SELECT
        Diagnosis,
        COUNT(*) AS "Total Prescriptions"
    FROM
        medications
    WHERE
        start >= '2015-01-01'
        AND Diagnosis <> ''
    GROUP BY
        Diagnosis
)
SELECT
	T.Diagnosis,
	M.MEDICATIONDESCRIPTION AS "Most Common Medication",
	COUNT(*) AS "Prescription Count",
	(COUNT(*) * 100.0 / TP."Total Prescriptions") AS "% of Total"
FROM
    	medications M
JOIN
    	TopDiagnoses T
ON
    	M.Diagnosis = T.Diagnosis
JOIN
    	TotalPrescriptions TP
ON
    	M.Diagnosis = TP.Diagnosis
WHERE
    	M.start >= '2015-01-01'
    	AND M.Diagnosis <> ''
GROUP BY
    	T.Diagnosis, M.MEDICATIONDESCRIPTION, TP."Total Prescriptions"
ORDER BY
    	T."Total Number of Prescriptions" DESC, "Prescription Count" DESC;
````

**Results:**

Diagnosis     |Total Number of Prescriptions| Most Common Medication | Prescription Count| % of Total| 
---------------|----------|--|--|--|
Acute Bronchitis    |    145| Acetaminophen 160 MG| 132 |91.0%|
Primary small cell malignant neoplasm of lung TNM stage 4 (disorder)    |     58| Cisplatin 50 MG Injection | 32 | 55.2%|
Viral sinusitis (disorder)  |     49| Amoxicillin 250 MG / Clavulanate 125 MG (Augmentin) | 47 |96.2%|
Coronary Heart Disease|   32| Nitroglycerin 0.4 MG/ACTUAT [Nitrolingual] |7 | 21.9%|
Streptococcal sore throat (disorder) |     30| Penicillin V Potassium 250 MG|         30|100%|

**8.** What is the highest, lowest and average Body Mass Index (BMI) Of all patients diagnosed with prediabetes after 2010? What percentage of prediabetes patients have BMI above 25 (overweight)?


````sql
SELECT
    SUBSTRING(START, 1, 4) AS year,
    MAX(CAST("VALUE" AS DOUBLE)) AS "Max BMI",
    MIN(CAST("VALUE" AS DOUBLE)) AS "Min BMI",
    AVG(CAST("VALUE" AS DOUBLE)) AS "Avg BMI",
    CONCAT(
        CAST(
            ROUND(
                (
                    COUNT(
                        DISTINCT CASE
                            WHEN CAST("VALUE" AS DOUBLE) > 25 THEN c.patient
                        END
                    )
                    * 100.0
                    / COUNT(DISTINCT c."PATIENT")
                ),
                2
            ) AS VARCHAR
        ),
        ' %'
    ) AS "% BMI > 25 (Overweight)"
FROM
	conditions AS c
FULL OUTER JOIN
	observations AS o
ON
	c.patient = o.patient
WHERE
	o.description = 'Body Mass Index'
	AND c.description LIKE '%prediabetes'
	AND year>'2010'
GROUP BY
	year
ORDER BY
	year 
````

**Results:**

year    |Max BMI|Min BMI|Avg BMI|% BMI > 25 (Overweight)|
---------|--------|-------------|----------------|---|
2011    |  52.28|       21.94|      34.16| 98.2%|
2012  | 40.13|        25.66|           32.77| 100%|
2013  | 39.2|         19.53|            31.21|80.0%|
2014     |  34.06|     14.12|         25.95| 71.43%|
2015| 45.13|     20.32|        31.21| 83.33%|
2016     | 38.57|       23.2|    30.38|   95.3%|
2017 |  38.33|       24.95|   32.11| 99.5%|


**9.** List the number and percentage of normal pregnacies that ended in a Cesarean section for the last 10 years.

````sql
SELECT
	year(date) as YEAR,
	count(CASE WHEN "DESCRIPTION" = 'Cesarean section' THEN 1 END) AS "Number of C-sections",
	count(CASE WHEN "REASONDESCRIPTION" LIKE '%Normal pregnancy' THEN 1 END) AS "Number of Normal Pregnancies",
    	format("Number of C-sections"/"Number of Normal Pregnancies",'P0') "% of C-sections"
FROM
 	procedures
GROUP BY
	YEAR
ORDER BY
	YEAR asc
LIMIT
	10;
````

**Results:**

Year   |Number of C-sections|Number of Normal Pregnancies|% of C-sections|
---------|--------|-------------|----------------|
2008     |  4|         76|           5.3%|
2009|     1|         59|            1.7%|
2010   |     5|         70|            7.1%|
2011     |    2 |         53|            3.8%|
2012 |     5|        83|            6.0%|
2013    |     1|         84|            1.2%|
2014 |     5|         65|            7.7%|
2015 |     5|         66|            7.6%|
2016 |     3|         63|            4.8%|
2017 |     5|         52|            9.6%|

**10.** For the last 10 years, what was the distribution of race among prediabetes patients? 

````sql
SELECT * FROM (
    SELECT
        EXTRACT(YEAR FROM c.start) AS YEAR,
        CAST(
            (
                round(COUNT(DISTINCT CASE WHEN p.race = 'black' THEN c.patient END)
                * 100.0
                / COUNT(DISTINCT c.patient),1)
            ) AS VARCHAR(20)
        )
        || '%' AS "% Black",
        CAST(
            (
                round(COUNT(DISTINCT CASE WHEN p.race = 'white' THEN c.patient END)
                * 100.0
                / COUNT(DISTINCT c.patient),1)
            ) AS VARCHAR(20)
        )
        || '%' AS "% White",
        CAST(
            (
                round(COUNT(DISTINCT CASE WHEN p.race = 'hispanic' THEN c.patient END)
                * 100.0
                / COUNT(DISTINCT c.patient),1)
            ) AS VARCHAR(20)
        )
        || '%' AS "% Hispanic",
        CAST(
            (
                round(COUNT(DISTINCT CASE WHEN p.race = 'asian' THEN c.patient END)
                * 100.0
                / COUNT(DISTINCT c.patient),1)
            ) AS VARCHAR(20)
        )
        || '%' AS "% Asian",


    FROM patients AS p INNER JOIN conditions AS c
        ON p.patient = c.patient
    WHERE c.description = 'Prediabetes'
    GROUP BY YEAR
    ORDER BY YEAR DESC
    LIMIT 10
)
ORDER BY YEAR ASC

````

**Results:**
YEAR|% Black|% White|% Hispanic|% Asian|
----|--------|-------|---|---|
2008|0.0%|78.6%|14.3%|7.1%|
2009|0.0%|75.0%|25.0|0.0%|
2010|0.0%|50%|37.5%|12.5%|
2011|10.0%|70.0%|20.0%|0.0%|
2012|0.0%|66.7%|0.0%|33.3%|
2013|20.0%|80.0%|0.0%|0.0%|
2014|14.3%|66.7%|2.3%|16.7%|
2015|16.7%|66.7%|0.0%|16.7%|
2016| 0.0%|50.0%|0.0%|50.0%|
2017|50.0%|50.0%|0.0%|0.0%|

