# Impact-Analysis-of-GoodThought-NGO-Initiatives
Analyzing GoodThought NGO’s initiatives (2010–2023) to measure funding flow and on-ground impact across regions and assignments; identifying top-funded projects by donor type and highest-impact assignments per region with confirmed donations, enabling data-driven prioritization of resources and program scaling. and improving beneficiary outcomes.
<br>
<br>

## PROJECT OVERVIEW:
GoodThought NGO has been actively working to create positive change by supporting underprivileged communities through education, healthcare, and sustainable development initiatives. This project provides a hands-on opportunity to analyze the NGO’s data using the GoodThought PostgreSQL database, which contains detailed records of assignments, donations, and donor information from 2010 to 2023.

The database includes three main tables:
* Assignments – Contains information about each project, such as the name, start and end dates, budget, geographical region, and the impact score, which measures the effectiveness of the assignment.
* Donations – Captures records of financial contributions, linking donors to specific assignments and showing how funding is distributed and utilized.
* Donors – Provides details of individuals and organizations that fund the NGO’s initiatives, including their donor type, which helps categorize and understand the source of support.

Through this project, we aim to explore how financial contributions and project outcomes are linked and determine which assignments make the highest impact in different regions. By analyzing this data, stakeholders can gain actionable insights to improve planning, resource allocation, and maximize the benefits of GoodThought’s initiatives.
<br>
<br>

## OBJECTIVE / KEY QUESTIONS

* List the top five assignments based on the total value of donations, categorized by donor type.
* Identify the assignment with the highest impact score in each region, ensuring each listed assignment has received at least one donation.
<br>

## DATASET

* **Source**: GoodThought PostgreSQL database (internal program records, 2010–2023)
* **Tables**:
    - `assignments`
    - `donations`
    - `donors`
* **Structure**:
  
  A. `aasignments` Table
  
     | Column Name       | Data Type      | Description                     |
     |-------------------|----------------|---------------------------------|
     | assignment_id     | integer        | Unique id for each assignment   |
     | assignment_name   | varchar        | Name of the assignment/project  |
     | start_date        | varchar / date | Start of assignment             |
     | end_date          | varchar / date | End of assignment               |
     | budget            | decimal        | Planned budget                  |
     | region            | varchar        | Geographic region               |
     | impact_score      | decimal        | Impact metric for assignment    |
  <br>
  
  B. `donations` Table
  
     | Column Name   | Data Type      | Description              |
     |---------------|----------------|--------------------------|
     | donation_id   | integer        | Unique id for donation   |
     | donor_id      | integer        | Links to donors          |
     | amount        | decimal        | Donation amount          |
     | donation_date | text / date    | When donation was made   |
     | assignment_id | integer        | Links to assignments     |
     | status        | varchar        | e.g., confirmed, pending |
     | created_at    | timestamp      | Record created           |
  <br>
  
  C. `donors` Table
     | Column Name | Data Type | Description                             |
     |-------------|-----------|-----------------------------------------|
     | donor_id    | integer   | Unique id for donor                     |
     | donor_name  | varchar   | Name of donor                           |
     | donor_type  | varchar   | individual, foundation, corporate, etc. |

* **ERD Daigram:** 
* **Sample preview:**
  
 ```
sql

SELECT *
FROM assignments
LIMIT 10;
```
<img width="1177" height="434" alt="image" src="https://github.com/user-attachments/assets/4b248f8d-ca30-4064-92d2-5387c7b27f74" /> 

```
sql

SELECT *
FROM donations
LIMIT 10;
```
<img width="1176" height="419" alt="image" src="https://github.com/user-attachments/assets/da5a6f3d-689e-430a-a6de-06da578ee078" /> 

```
sql

SELECT *
FROM donors
LIMIT 10;
```
<img width="1164" height="423" alt="image" src="https://github.com/user-attachments/assets/2815ab3e-af9f-4b21-af73-1a2d1dd1134e" />

Looking at the first 10 rows of each table tells us several important things before deep analysis: it confirms the presence and format of key fields (IDs, names, amounts, dates), shows whether values look sensible (e.g., donation amounts are positive and non-zero), and reveals immediate data quality issues like missing assignment IDs or unexpected status values. From this small snapshot, we can detect if donations link to assignments and donors correctly, if region names are consistent, and if impact scores are populated. That early check prevents misleading conclusions later — for example, we avoid ranking an assignment highly when it simply has many null donations or mis-typed donor IDs. In short, the preview is a practical sanity check that shapes accurate, trustworthy analysis. 
<br>
<br>

## TOOLS AND SKILLS USED

* **SQL (PostgreSQL) concepts**: Common Table Expressions / `WITH (CTE), SUM() and ROUND(), GROUP BY, INNER JOIN`, Window functions `ROW_NUMBER()` with `PARTITION BY and ORDER BY, COUNT(), ORDER BY, LIMIT`
* **Techniques used:** 
  - Data exploration and validation (preview tables, metadata)
  - Aggregation of monetary flows (summing donations per assignment and donor)
  - Rounding and presentation of monetary totals for readability
  - Segmenting results by donor type to compare funding sources
  - Ensuring only assignments with actual donations are considered (filter by donation counts)
  - Ranking by impact within regions to pick the top assignment per region
<br>
<br>

## ANALYSIS & APPROACH

### 1. Exploring the data

```
sql

SELECT * FROM assignments;
```
<img width="1179" height="627" alt="image" src="https://github.com/user-attachments/assets/bdb698c6-f611-479b-82c3-7efbaff00d82" />

```
sql

SELECT * FROM donations;
```
<img width="1181" height="627" alt="image" src="https://github.com/user-attachments/assets/8d1404fd-320f-4835-819b-27aae70a4bcd" />

```
sql

SELECT * FROM donors;
```
<img width="1179" height="632" alt="image" src="https://github.com/user-attachments/assets/0e5c44bb-2289-444c-82e6-357fe32aa7c4" />

These queries fetch the first few rows of each table (`assignments`, `donations`, and `donors`). The purpose is to get an initial understanding of the data—such as what information is available, the types of values stored, and whether there are any obvious missing or unusual entries. Previewing the data helps in planning the analysis by showing how assignments are linked to donations and donors, which is essential for combining tables later in queries. It provides a real-world view of what we are working with before performing calculations or aggregations.






































