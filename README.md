<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=21&pause=1200&color=4285F4&center=true&vCenter=true&width=760&lines=BigQuery+Challenge+Lab;Learn+SQL+by+Solving+Real+Questions;Explore+COVID-19+Open+Data" alt="BigQuery Challenge Lab typing animation" />
</p>

# 🚀 Derive Insights from BigQuery Data: Challenge Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Google%20Skills-4285F4?logo=googlecloud&logoColor=white" alt="Google Skills" />
  <img src="https://img.shields.io/badge/Topic-BigQuery%20SQL-669DF6?logo=googlebigquery&logoColor=white" alt="BigQuery SQL" />
  <img src="https://img.shields.io/badge/Purpose-Educational-2E8B57" alt="Educational purpose" />
</p>

**Official learning platform:** [Google Skills](https://www.skills.google/)

------------------------------------------------------------------------

## ⚠️ Disclaimer

This repository is an independent set of personal learning notes created
for **educational and practice purposes**. It is intended to help
learners understand BigQuery SQL concepts through examples and short
explanations.

-   **Official platform:** The original lab experience and learning materials are provided through [Google Skills](https://www.skills.google/).
-   **Respect platform rules:** Complete the lab yourself and follow
    Google Skills/Qwiklabs terms, assessment rules, and applicable
    service policies. These notes are not intended to bypass lab
    requirements or assessments.
-   **Personal contribution:** The queries and explanations below are my
    learning notes. They are not an official Google solution or an
    endorsement by Google.
-   **Data note:** COVID-19 figures are historical dataset values and
    may differ from other sources because of data definitions,
    reporting, or later revisions.

## 🎯 What this repository covers

-   Practising SQL with Google BigQuery.
-   Using `SUM`, `COUNT`, `GROUP BY`, `HAVING`, and `ORDER BY`.
-   Comparing daily totals with `LAG()` and `LEAD()`.
-   Calculating growth, recovery, and fatality ratios.
-   Preparing query output for a simple report.

## 🧪 Dataset used

``` sql
`bigquery-public-data.covid19_open_data.covid19_open_data`
```

The examples use the public COVID-19 open-data table. Queries are
written as learning examples; check the dataset schema and run them in
BigQuery to confirm results in your environment.

------------------------------------------------------------------------

## 📌 Task 1. Total confirmed cases

**Question:** What was the total count of confirmed cases worldwide on
**June 25, 2020**? Return one row with the column
`total_cases_worldwide`.

**Columns:** `cumulative_confirmed`, `date`

``` sql
SELECT
  SUM(cumulative_confirmed) AS total_cases_worldwide
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date = '2020-06-25';
```

**Simple explanation:** `WHERE` selects records for June 25, 2020.
`SUM()` adds the confirmed-case values across the selected records, and
`AS` gives the result the requested column name.

------------------------------------------------------------------------

## 📍 Task 2. Worst affected areas

**Question:** How many US states had more than **300 deaths** on **June
25, 2020**? Return the count as `count_of_states` and exclude missing
state names.

**Columns:** `country_name`, `subregion1_name`, `cumulative_deceased`

``` sql
SELECT
  COUNT(*) AS count_of_states
FROM (
  SELECT
    subregion1_name,
    SUM(cumulative_deceased) AS total_deaths
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'United States of America'
    AND date = '2020-06-25'
    AND subregion1_name IS NOT NULL
  GROUP BY
    subregion1_name
  HAVING
    SUM(cumulative_deceased) > 300
);
```

**Simple explanation:** The inner query groups records by state and
calculates deaths for the selected date. `HAVING` keeps only states with
more than 300 deaths. The outer `COUNT(*)` counts those states.

------------------------------------------------------------------------

## 🔥 Task 3. Identify hotspots

**Question:** List US states with more than **2,500 confirmed cases** on
**June 25, 2020**, showing the state and case count in descending order.

**Output fields:** `state`, `total_confirmed_cases`

**Columns:** `country_code`, `subregion1_name`, `cumulative_confirmed`

``` sql
SELECT
  subregion1_name AS state,
  SUM(cumulative_confirmed) AS total_confirmed_cases
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_code = 'US'
  AND date = '2020-06-25'
  AND subregion1_name IS NOT NULL
GROUP BY
  subregion1_name
HAVING
  SUM(cumulative_confirmed) > 2500
ORDER BY
  total_confirmed_cases DESC;
```

**Simple explanation:** The query groups records by state, adds the
confirmed cases for each state, and filters groups above 2,500 using
`HAVING`. `ORDER BY ... DESC` lists the states from the highest count to
the lowest.

------------------------------------------------------------------------

## ⚖️ Task 4. Fatality ratio

**Question:** What was Italy's case-fatality ratio for **May 2020**?

The ratio is:

`(total deaths / total confirmed cases) × 100`

**Output fields:** `total_confirmed_cases`, `total_deaths`,
`case_fatality_ratio`

**Columns:** `country_name`, `cumulative_confirmed`,
`cumulative_deceased`

``` sql
SELECT
  SUM(cumulative_confirmed) AS total_confirmed_cases,
  SUM(cumulative_deceased) AS total_deaths,
  SAFE_DIVIDE(
    SUM(cumulative_deceased),
    SUM(cumulative_confirmed)
  ) * 100 AS case_fatality_ratio
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name = 'Italy'
  AND date BETWEEN '2020-05-01' AND '2020-05-31';
```

**Simple explanation:** The query filters Italy's records to May 2020,
sums confirmed cases and deaths, then calculates the percentage of
confirmed cases represented by deaths. `SAFE_DIVIDE` avoids a
division-by-zero error.

**Learning note:** This follows the requested exercise formula over the
selected date records. Since cumulative figures can be repeated across
geographic rows, check the dataset's granularity when interpreting the
totals.

------------------------------------------------------------------------

## 📅 Task 5. Identify a specific day

**Question:** On what date did Italy's total deaths first exceed
**8,000**? Return the date in `yyyy-mm-dd` format.

**Columns:** `country_name`, `cumulative_deceased`

``` sql
SELECT
  date
FROM (
  SELECT
    date,
    SUM(cumulative_deceased) AS total_deaths
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'Italy'
  GROUP BY
    date
)
WHERE
  total_deaths > 8000
ORDER BY
  date ASC
LIMIT 1;
```

**Simple explanation:** The inner query calculates Italy's death total
for each date. The outer query keeps dates where the total exceeds
8,000, sorts from earliest to latest, and returns the first matching
date.

------------------------------------------------------------------------

## 🇮🇳 Task 6. Find days with zero net new cases

**Question:** Between February 24 and March 14, 2020, how many days in
India had no increase in confirmed cases compared with the previous day?

``` sql
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'India'
    AND date BETWEEN '2020-02-24' AND '2020-03-14'
  GROUP BY
    date
),
india_previous_day_comparison AS (
  SELECT
    date,
    cases,
    LAG(cases) OVER (ORDER BY date) AS previous_day,
    cases - LAG(cases) OVER (ORDER BY date) AS net_new_cases
  FROM
    india_cases_by_date
)
SELECT
  COUNT(*) AS days_with_zero_net_new_cases
FROM
  india_previous_day_comparison
WHERE
  net_new_cases = 0;
```

**Simple explanation:** The first CTE calculates confirmed cases for
each day. `LAG()` retrieves the previous day's total, and subtracting it
gives the net change. The final query counts days where that change is
zero. The first date has no previous date in this range, so it is not
counted.

------------------------------------------------------------------------

## 📈 Task 7. Doubling rate --- daily percentage increase

**Question:** List dates from **March 22 to April 20, 2020** when US
confirmed cases increased by more than **15%** compared with the
previous day. Return the date, current cases, previous-day cases, and
percentage increase.

**Output fields:** `Date`, `Confirmed_Cases_On_Day`,
`Confirmed_Cases_Previous_Day`, `Percentage_Increase_In_Cases`

``` sql
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'United States of America'
    AND date BETWEEN '2020-03-21' AND '2020-04-20'
  GROUP BY
    date
),
us_previous_day_comparison AS (
  SELECT
    date,
    cases,
    LAG(cases) OVER (ORDER BY date) AS previous_day,
    SAFE_DIVIDE(
      cases - LAG(cases) OVER (ORDER BY date),
      LAG(cases) OVER (ORDER BY date)
    ) * 100 AS percentage_increase
  FROM
    us_cases_by_date
)
SELECT
  date AS Date,
  cases AS Confirmed_Cases_On_Day,
  previous_day AS Confirmed_Cases_Previous_Day,
  percentage_increase AS Percentage_Increase_In_Cases
FROM
  us_previous_day_comparison
WHERE
  date BETWEEN '2020-03-22' AND '2020-04-20'
  AND percentage_increase > 15
ORDER BY
  date ASC;
```

**Simple explanation:** The query gets daily US totals, uses `LAG()` to
find the previous day's cases, and calculates the percentage increase.
It includes the day before the requested output range so March 22 can be
compared with March 21. The final filter returns increases above 15%.

**Note:** A daily increase above 15% is the exercise's threshold; it is
not an exact guarantee that cases will double in seven days.

------------------------------------------------------------------------

## 💚 Task 8. Recovery rate

**Question:** On **May 10, 2020**, list the 15 countries with the
highest recovery rates among countries with more than **50,000 confirmed
cases**.

**Output fields:** `country`, `recovered_cases`, `confirmed_cases`,
`recovery_rate`

**Columns:** `country_name`, `cumulative_confirmed`,
`cumulative_recovered`

``` sql
SELECT
  country_name AS country,
  SUM(cumulative_recovered) AS recovered_cases,
  SUM(cumulative_confirmed) AS confirmed_cases,
  SAFE_DIVIDE(
    SUM(cumulative_recovered),
    SUM(cumulative_confirmed)
  ) * 100 AS recovery_rate
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date = '2020-05-10'
  AND country_name IS NOT NULL
GROUP BY
  country_name
HAVING
  SUM(cumulative_confirmed) > 50000
ORDER BY
  recovery_rate DESC
LIMIT 15;
```

**Simple explanation:** The query calculates each country's recovered
and confirmed totals for May 10, then computes the recovery percentage.
`HAVING` keeps countries above 50,000 confirmed cases, and `ORDER BY`
plus `LIMIT 15` returns the top 15 recovery rates.

------------------------------------------------------------------------

## 📊 Task 9. CDGR --- Cumulative daily growth rate

**Question:** Calculate France's cumulative daily growth rate from the
first reported case date, **January 24, 2020**, to **June 15, 2020**.

The formula is:

`CDGR = (last_day_cases / first_day_cases)^(1 / days_diff) - 1`

``` sql
WITH france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'France'
    AND date IN ('2020-01-24', '2020-06-15')
  GROUP BY
    date
),
summary AS (
  SELECT
    total_cases AS first_day_cases,
    LEAD(total_cases) OVER (ORDER BY date) AS last_day_cases,
    DATE_DIFF(
      LEAD(date) OVER (ORDER BY date),
      date,
      DAY
    ) AS days_diff
  FROM
    france_cases
)
SELECT
  first_day_cases,
  last_day_cases,
  days_diff,
  POWER(
    SAFE_DIVIDE(CAST(last_day_cases AS FLOAT64), first_day_cases),
    SAFE_DIVIDE(1.0, days_diff)
  ) - 1 AS cdgr
FROM
  summary
WHERE
  last_day_cases IS NOT NULL;
```

**Simple explanation:** The first CTE gets France's confirmed-case
totals on the two dates. `LEAD()` retrieves the later total and date,
while `DATE_DIFF()` calculates the number of days between them.
`POWER()` applies the CDGR formula.

**Note:** CDGR is returned as a decimal rate. Multiply it by 100 if you
want to express it as a percentage.

------------------------------------------------------------------------

## 📉 Task 10. Create a Data Studio report

**Goal:** Create a report for the United States showing confirmed cases
and deaths from **March 30 to April 24, 2020**.

1.  Open [Google Data Studio / Looker
    Studio](https://datastudio.google.com/).
2.  Create a report and select the BigQuery connector.
3.  Authorize access if prompted.
4.  Choose **Custom Query** under your project, then enter the SQL
    below.
5.  Click **Add**, then **Add to report**.
6.  Add a time-series chart using `date` as the date dimension and
    `confirmed_cases` and `deaths` as metrics.

``` sql
SELECT
  date,
  SUM(cumulative_confirmed) AS confirmed_cases,
  SUM(cumulative_deceased) AS deaths
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name = 'United States of America'
  AND date BETWEEN '2020-03-30' AND '2020-04-24'
GROUP BY
  date
ORDER BY
  date ASC;
```

**Simple explanation:** The query produces one row per date with the US
confirmed-case and death totals. The report can plot both metrics over
time to make changes easier to compare.

**Note:** Google Data Studio is now called Looker Studio. The interface
and project-specific instructions may differ from the original lab.

------------------------------------------------------------------------

## 🙏 Credits and acknowledgement

-   **Google Skills:** The official learning platform for the lab and
    related training resources --- <https://www.skills.google/>
-   **Google Cloud BigQuery:** The SQL environment and public dataset
    used for practice --- <https://cloud.google.com/bigquery>
-   **COVID-19 Open Data:** The public dataset referenced by the queries
    in this README.

This repository is an independent educational resource. Google, Google
Cloud, and the respective content owners retain ownership of their
names, platforms, and original learning materials. This README is not an
official Google Skills publication.

------------------------------------------------------------------------
---

<p align="center">
  <strong>✨ Learn • Practise • Understand • Share responsibly ✨</strong>
</p>
