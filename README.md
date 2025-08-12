# Layoffs-Lens-SQL-EDA-
Layoffs Lens is a MySQL-based exploratory data analysis of a layoffs dataset. It surfaces headline metrics, yearly and monthly totals with a rolling cumulative, and ranks the top impacted companies each year using window functions. Built for quick insight and easy charting.

## ✨ Highlights
- Headline maxima (`MAX`) for layoffs and percentage laid off
- Companies with 100% layoffs, sorted by funding
- Yearly totals and monthly totals with a rolling cumulative
- Top companies overall and **Top 5 per year** via `DENSE_RANK`
- MySQL 8+ compatible (uses window functions)

---

## 🧰 Tech & Requirements
- **Database:** MySQL **8.0+**
- **Primary table:** `layoffs_staging2`

### Expected columns
| column                  | type (recommended) | notes                              |
|-------------------------|--------------------|------------------------------------|
| `company`               | TEXT/VARCHAR       | company name                       |
| `total_laid_off`        | INT                | count of laid off employees        |
| `percentage_laid_off`   | DECIMAL(5,4)       | 0–1 (e.g., 1.0 = 100%)             |
| `funds_raised_millions` | DECIMAL/FLOAT/INT  | funding in USD millions            |
| `date`                  | **DATE**           | coverage date                      |

> If `date` was loaded as text, convert it:
```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%Y-%m-%d');
