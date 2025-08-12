# Layoffs-Lens-SQL-EDA-
Layoffs Lens is a MySQL-based exploratory data analysis of a layoffs dataset. It surfaces headline metrics, yearly and monthly totals with a rolling cumulative, and ranks the top impacted companies each year using window functions. Built for quick insight and easy charting.

Overview
Layoffs Lens (SQL EDA) is a MySQL-based exploratory data analysis of a layoffs dataset.
It surfaces headline metrics, yearly and monthly trends (with a rolling cumulative), and the top impacted companies overall and per year.

Tech & Requirements
Database: MySQL 8.0+ (uses window functions like DENSE_RANK and SUM() OVER)

Primary table: layoffs_staging2

Expected columns used
company (TEXT)

total_laid_off (INT)

percentage_laid_off (DECIMAL or FLOAT; values 0–1)

funds_raised_millions (DECIMAL/FLOAT/INT)

date (DATE or text representing a date like YYYY-MM-DD)

If date is stored as text, convert it on load:

sql
Copy
Edit
-- Example: convert text date -> DATE
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%Y-%m-%d');
Prefer DATE_FORMAT(date,'%Y-%m') over SUBSTRING(date,1,7) once date is a real DATE.

What’s inside (key queries)
Initial scan

sql
Copy
Edit
SELECT * FROM layoffs_staging2;
Headline maxima

sql
Copy
Edit
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
Companies with 100% layoffs (sorted by funding)

sql
Copy
Edit
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
Most impacted companies (overall)

sql
Copy
Edit
SELECT company, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
GROUP BY company
ORDER BY total_off DESC;
Date coverage

sql
Copy
Edit
SELECT MIN(`date`) AS first_date, MAX(`date`) AS last_date
FROM layoffs_staging2;
Yearly totals

sql
Copy
Edit
SELECT YEAR(`date`) AS yr, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY yr DESC;
Monthly totals + rolling cumulative

sql
Copy
Edit
-- If `date` is DATE:
WITH monthly AS (
  SELECT DATE_FORMAT(`date`, '%Y-%m') AS `month`,
         SUM(total_laid_off) AS total_off
  FROM layoffs_staging2
  GROUP BY 1
)
SELECT `month`,
       total_off,
       SUM(total_off) OVER (ORDER BY `month`) AS rolling_total
FROM monthly
ORDER BY `month` ASC;
Per-company, per-year totals

sql
Copy
Edit
SELECT company, YEAR(`date`) AS yr, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY total_off DESC;
Top 5 companies by layoffs each year (dense rank)

sql
Copy
Edit
WITH company_year AS (
  SELECT company,
         YEAR(`date`) AS yrs,
         SUM(total_laid_off) AS total_off
  FROM layoffs_staging2
  GROUP BY company, YEAR(`date`)
),
ranked AS (
  SELECT *,
         DENSE_RANK() OVER (PARTITION BY yrs ORDER BY total_off DESC) AS ranking
  FROM company_year
  WHERE yrs IS NOT NULL
)
SELECT *
FROM ranked
WHERE ranking <= 5
ORDER BY yrs DESC, ranking ASC;
How to run
Ensure MySQL 8.0+ is installed and your data is loaded into layoffs_staging2.

Confirm date is DATE. If not, run the conversion snippet above.

Execute the queries in order (they’re standalone, so you can also run the sections you need).

What you’ll learn/produce
Scope: dataset date range and basic completeness checks.

Scale: max layoffs and companies fully laid off (100%).

Trends: yearly totals, monthly totals, and a rolling cumulative line you can chart.

Leaders: overall most-impacted companies and top 5 per year via DENSE_RANK.

Visualization ideas
Yearly totals: column chart (year vs. sum).

Monthly + rolling: line chart (month vs. monthly sum) + line for rolling cumulative.

Top 5 per year: small multiples or a faceted bar chart by year.

Notes & assumptions
Nulls in total_laid_off / date should be handled as needed (e.g., filter out or impute).

percentage_laid_off is assumed in the 0–1 range; multiply by 100 for % display.

funds_raised_millions is assumed numeric; cast if it was ingested as text.
