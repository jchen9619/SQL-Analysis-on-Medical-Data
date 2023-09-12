# SQL Analysis on Medical Data

**Author**: Chen Chen <br />
**Email**: cc4865@nyu.edu <br />
**Website**: chencapp.github.io <br />
**LinkedIn**: www.linkedin.com/in/cchengwnyu <br />

## Introduction
An SQL analysis of patient demographics, medical conditions, prescriptions, biometrics and disease prevalences from 1919 to .  The data was genererated using Synthea, a synthetic patient generator that models the medical history of synthetic patients. Their mission is to output high-quality synthetic, realistic but not real, patient data and associated health records covering every aspect of healthcare. 

* [Data Analysis Question & Answers](https://github.com/jchen9619/SQL-Analysis-on-Medical-Data/blob/main/Q%26A_Analysis.md)

## Datasets used
Seven key [datasets](https://github.com/jchen9619/SQL-Analysis-on-Medical-Data/tree/main/data/csv) for this case study
- <strong>patients.csv</strong>: Demographic information of patients including marital staus, race, ethnicity, birthplace and address.
- <strong>all_prevalences.csv</strong>: Prevalence rate and percentage of conditions and medications.
- <strong>encounters.csv</strong>: Type ad reason for each visit, organized by date.
- <strong>conditions.csv</strong>: Medical conditions diagnosed at each encounter.
- <strong>medications.csv</strong>: Drugs prescribed at each encounter .
- <strong>immunizations.csv</strong>: Patient record or immunizations.
- <strong>procedures.csv</strong>: Surgical and non-surgical procedures performed for different diagnosis.
- <strong>observations.csv</strong>: Health metrics such as blood pressure, height, BMI.

## Entity Relationship Diagram
![alt text](https://github.com/jchen9619/SQL-Analysis-on-Medical-Data/blob/main/images/ERD.png)
