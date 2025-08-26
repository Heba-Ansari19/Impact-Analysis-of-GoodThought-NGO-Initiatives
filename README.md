# Impact-Analysis-of-GoodThought-NGO-Initiatives
Analyzing GoodThought NGO’s initiatives (2010–2023) to measure funding flow and on-ground impact across regions and assignments; identifying top-funded projects by donor type and highest-impact assignments per region with confirmed donations, enabling data-driven prioritization of resources and program scaling. and improving beneficiary outcomes.
<br>
<br>

## Project Overview

GoodThought NGO has been actively working to create positive change by supporting underprivileged communities through education, healthcare, and sustainable development initiatives. This project provides a hands-on opportunity to analyze the NGO’s data using the GoodThought PostgreSQL database, which contains detailed records of assignments, donations, and donor information from 2010 to 2023.

The database includes three main tables:
* Assignments – Contains information about each project, such as the name, start and end dates, budget, geographical region, and the impact score, which measures the effectiveness of the assignment.
* Donations – Captures records of financial contributions, linking donors to specific assignments and showing how funding is distributed and utilized.
* Donors – Provides details of individuals and organizations that fund the NGO’s initiatives, including their donor type, which helps categorize and understand the source of support.

Through this project, we aim to explore how financial contributions and project outcomes are linked and determine which assignments make the highest impact in different regions. By analyzing this data, stakeholders can gain actionable insights to improve planning, resource allocation, and maximize the benefits of GoodThought’s initiatives.

<img width="500" height="353" alt="image" src="https://github.com/user-attachments/assets/42a9c785-5a97-413e-8c62-5f595b651460" />

<br>
<br>

## Objective / Key Questions

* List the top five assignments based on the total value of donations, categorized by donor type.
* Identify the assignment with the highest impact score in each region, ensuring each listed assignment has received at least one donation.
<br>

## Dataset

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
  
  C. `donors` Table
     | Column Name | Data Type | Description                             |
     |-------------|-----------|-----------------------------------------|
     | donor_id    | integer   | Unique id for donor                     |
     | donor_name  | varchar   | Name of donor                           |
     | donor_type  | varchar   | individual, foundation, corporate, etc. |

* **ERD Diagram:** <img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/635790c7-1ce6-48a4-a82a-a71567c1124b" />

* **Sample preview:**
  
 ```
sql

SELECT *
FROM assignments
LIMIT 10;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/4b248f8d-ca30-4064-92d2-5387c7b27f74" /> 


```
sql

SELECT *
FROM donations
LIMIT 10;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/da5a6f3d-689e-430a-a6de-06da578ee078" /> 


```
sql

SELECT *
FROM donors
LIMIT 10;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/2815ab3e-af9f-4b21-af73-1a2d1dd1134e" />


Looking at the first 10 rows of each table tells us several important things before deep analysis: it confirms the presence and format of key fields (IDs, names, amounts, dates), shows whether values look sensible (e.g., donation amounts are positive and non-zero), and reveals immediate data quality issues like missing assignment IDs or unexpected status values. From this small snapshot, we can determine if donations are linked to assignments and donors correctly, if region names are consistent, and if impact scores are populated. That early check prevents misleading conclusions later — for example, we avoid ranking an assignment highly when it simply has many null donations or mis-typed donor IDs. In short, the preview is a practical sanity check that shapes accurate, trustworthy analysis. 
<br>
<br>

## Tools & Skills Used

* **SQL (PostgreSQL) concepts**: Common Table Expressions / `WITH (CTE), SUM() and ROUND(), GROUP BY, INNER JOIN`, Window functions `ROW_NUMBER()` with `PARTITION BY and ORDER BY, COUNT(), ORDER BY, LIMIT`
* **Techniques used:** 
  - Data exploration and validation (preview tables, metadata)
  - Aggregation of monetary flows (summing donations per assignment and donor)
  - Rounding and presentation of monetary totals for readability
  - Segmenting results by donor type to compare funding sources
  - Ensuring only assignments with actual donations are considered (filter by donation counts)
  - Ranking by impact within regions to pick the top assignment per region
<br>

## Analysis & Approach

### 1. Exploring the data

```
sql

SELECT * FROM assignments;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/bdb698c6-f611-479b-82c3-7efbaff00d82" />


```
sql

SELECT * FROM donations;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/8d1404fd-320f-4835-819b-27aae70a4bcd" />


```
sql

SELECT * FROM donors;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/0e5c44bb-2289-444c-82e6-357fe32aa7c4" />

These queries fetch the first few rows of each table (`assignments`, `donations`, and `donors`). The purpose is to get an initial understanding of the data, such as what information is available, the types of values stored, and whether there are any obvious missing or unusual entries. Previewing the data helps in planning the analysis by showing how assignments are linked to donations and donors, which is essential for combining tables later in queries. It provides a real-world view of what we are working with before performing calculations or aggregations.



### 2. Checking Table Structure and Column Details

```
sql

SELECT 
    table_name, 
    column_name, 
    data_type, 
    is_nullable,
    character_maximum_length as max_len 
FROM information_schema.columns 
WHERE table_name IN ('assignments', 'donations', 'donors') 
ORDER BY table_name, ordinal_position;
```
This query examines the structure of the three tables by listing each column’s name, data type, whether it allows null values, and its maximum length (for text columns). Understanding the structure is crucial for accurate analysis because it informs us about:
- **Relationships**: Which columns can be used to join tables (e.g., assignment_id, donor_id).
- **Data types**: Which columns are numeric, text, or dates, helping us choose the correct aggregation or calculation methods.
- **Constraints**: Which columns are mandatory (NOT NULL) and which may have missing values, helping to anticipate data cleaning needs.
By analyzing table structures, we ensure that subsequent queries and calculations are reliable and accurate, thereby reducing the risk of errors and enhancing the quality of insights.

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/e3de15a3-8c73-46f2-bc38-ccbce774a8cd" />



### 3. Top Five Assignments Based on Total Donations by Donor Type

```
sql

-- highest_donation_assignments

WITH cte AS (
    SELECT 
        assignment_id,
        donor_id,
        ROUND(SUM(amount), 2) AS rounded_total_donation_amount
    FROM donations
    GROUP BY 
        assignment_id,
        donor_id
)

SELECT
    a.assignment_name,
    a.region,
    ROUND(SUM(cte.rounded_total_donation_amount), 2) AS rounded_total_donation_amount,
    d.donor_type
FROM assignments AS a
INNER JOIN cte 
    ON a.assignment_id = cte.assignment_id
INNER JOIN donors AS d 
    ON cte.donor_id = d.donor_id
GROUP BY 
    a.assignment_name,
    a.region,
    d.donor_type
ORDER BY rounded_total_donation_amount DESC
LIMIT 5;
```
* Goal connection: The question asks for assignments that received the most money, and it wants that view broken down by donor type. That means we must both total donations and keep track of who gave them. The CTE first groups donations by assignment and donor to get a donor-level contribution for each assignment; this prevents double-counting when donors give multiple times and keeps the donor breakdown clear. Rounding makes dollar amounts easier to read and compare.
  
* Linking assignments and donor types: By joining the aggregated donations to assignments and donors, the result shows the assignment name, its region, the total money tied to it (summed across donors of a given type), and the donor type (for example, corporate vs foundation vs individual). This directly answers “which assignments got the most funding and from which kinds of donors?” — information that program managers need when deciding whether to cultivate particular donor types or replicate successful funding models.
  
* Why grouped by donor type matters: Two assignments might have similar total donations but very different donor mixes (e.g., one funded mainly by small individual donors and the other by a few large corporate grants). Knowing donor type clarifies funding stability and scalability: corporate grants may be larger but less reliable year-to-year; many individual donors might show broad community support. The query’s structure surfaces these strategic differences.
  
* How this guides insights: The output lets us pick the top five assignments by funding and immediately see which donor types are behind that funding. From there, we can conclude things like: “Assignment A is top-funded thanks to corporate donors, so pursue more corporate partnerships for similar projects,” or “Assignment B has diverse donor types, implying strong community buy-in.” The result is actionable for fundraising strategy and program scaling.

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/8d7b1165-bec8-4bb3-bbc4-ae76ea14d4d5" />



### 4. Assignment with Highest Impact Score per Region (with at least one donation)

```
sql

-- top_regional_impact_assignments
WITH cte1 AS (
    SELECT 
        assignment_id,
        COUNT(*) AS num_total_donations
    FROM donations
    GROUP BY assignment_id),
    
cte2 AS (
    SELECT 
        assignment_id,
        ROW_NUMBER() OVER(PARTITION BY region ORDER BY impact_score DESC) AS rank_n    
    FROM assignments
    GROUP BY assignment_id
    ORDER BY impact_score DESC
)

SELECT 
    a.assignment_name,
    a.region,
    a.impact_score,
    cte1.num_total_donations
FROM assignments AS a
INNER JOIN cte1 
    ON a.assignment_id = cte1.assignment_id
INNER JOIN cte2
    ON cte1.assignment_id = cte2.assignment_id
WHERE cte2.rank_n = 1
GROUP BY a.assignment_name, a.region, a.impact_score, cte1.num_total_donations
ORDER BY a.region ASC;
```
* Goal connection: The objective is to find the assignment with the highest impact score in each region, but only if that assignment has actually received donations. This ensures we focus on impactful work that is also funded — a practical combination for scaling or replication.

* Counting donations first (cte1): By counting donations per assignment up front, the analysis can later require num_total_donations to ensure the assignment has financial backing. This prevents recommending high-impact assignments that no one has funded (which could be great in theory, but unrealistic without support).

* Ranking by impact per region (cte2): Using a region-partitioned ranking finds the single highest-impact assignment per region. This isolates the top local performer rather than aggregating across regions, which is important because impact priorities differ by geography. For example, an education project may score very high in one region but not be relevant elsewhere. The partitioned ranking preserves that local view.

* Joining and filtering to ensure donations exist: By joining the donation counts to the assignments and then filtering to rank_n = 1, we get the highest-impact assignment in each region among those that have at least one donation. That ensures the output only lists assignments that both perform well and have demonstrated financial support — the most realistic candidates for further investment or expansion.

* How this guides insights: The result directly tells program leaders: “In region X, assignment Y has the top impact score and has received N donations.” That is powerful because it pairs performance evidence with funding reality. From this, we can recommend actions like increasing investment in that assignment, studying its model for replication, or engaging the assignment’s donors to scale successful interventions. It also highlights regions where top-impact assignments lack donations (if any), suggesting fundraising priorities.

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/34753bf3-5520-43f4-a398-e7859305ddb2" />
<br>
<br>

## Insights & Findings

* **Top-funded assignments reflect donor mix**: The highest donation totals often align with specific donor types; some top assignments receive most funds from corporate donors, while others rely on foundations or many individual donors. This helps GoodThought tailor fundraising outreach to the donor types that currently support the most impactful projects.
* **Funding vs impact alignment**: By requiring at least one donation before selecting top impact assignments by region, we ensure the recommended assignments are both effective and already supported; these are the strongest candidates for scaling because they combine measurable impact with demonstrated donor interest.
* **Regional leaders are actionable**: The assignment with the highest impact in each region provides a shortlist of “local champions” — projects that could be expanded or used as models for similar communities. For example, if a region’s top assignment focuses on school health and shows strong impact plus donations, it’s a natural candidate for additional funding.
* **Donation counts add context beyond totals**: Knowing both total donation amounts and the number of donations gives nuance: a high total driven by a single large donation differs in risk profile from similar funding coming from many smaller donors. GoodThought can use this to diversify revenue streams for risky projects.
* **PR and donor relations opportunities**: Assignments that are both high-impact and well-funded are strong stories for donor stewardship and public communications — promoting success helps retain donors and attract new ones.
* **Data checks revealed operational issues**: During previews and joins, we can detect mismatches (donations not linked to assignments, missing donor types), which indicate areas where data entry or process improvements would increase analysis reliability and donor reporting accuracy.
<br>

## Final Notes

This analysis transforms raw operational records into clear, practical guidance: which assignments are receiving the most funds (and from which donor types), and which assignments are proving to be the highest-impact options in each region with confirmed donations. These two views together let GoodThought prioritize funding, replicate the most successful approaches, and improve fundraising strategies by focusing on the donor types that matter.

Next steps GoodThought might take based on these findings include: strengthening relationships with donor types that fund successful assignments, exploring replication of regionally successful programs, and fixing any data linkage issues to make future analyses even more reliable. The structure used here ensures every recommendation is grounded in both impact evidence and funding reality.
<br>
