# EXPLORATORY DATA ANALYSIS
### PROJECT OVERVIEW
This is the exploratory data analysis of the dataset using MySQL server

### Dataset Source
[Download here](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

### Tools
- MySQL Server

### Data Analysis

```sql

SELECT MAX(total_laid_off), MAX(percentage_laid_off) FROM layoffs_static2;

SELECT * FROM layoffs_static2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;

SELECT company, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY company
ORDER BY 2 DESC;

SELECT MIN(`date`), MAX(`date`)
FROM layoffs_static2;

SELECT industry, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY industry
ORDER BY 2 DESC;

SELECT country, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY country
ORDER BY 2 DESC;

SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_static2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;

SELECT SUBSTRING(`date`, 1, 7) AS MONTH, SUM(total_laid_off)
FROM layoffs_static2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY MONTH
ORDER BY 1 ASC;

WITH rolling_total AS 
(
SELECT SUBSTRING(`date`, 1, 7) AS MONTH, SUM(total_laid_off) AS tlo
FROM layoffs_static2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY MONTH
ORDER BY 1 ASC
)
SELECT `MONTH`, tlo, SUM(tlo) OVER (ORDER BY `MONTH`) AS Rolling_Total
FROM rolling_total;

SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_static2
GROUP BY company, YEAR(`date`)
ORDER BY company ASC;

WITH company_year(compay, years, total_laid_off)AS 
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_static2
GROUP BY company, YEAR(`date`)
), company_year_ranking AS
(
SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM company_year
WHERE years IS NOT NULL
ORDER BY ranking ASC
)

SELECT * 
FROM company_year_ranking 
WHERE ranking <= 5
```
