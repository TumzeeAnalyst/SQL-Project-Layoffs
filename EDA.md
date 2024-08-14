# EXPLORATORY DATA ANALYSIS
### PROJECT OVERVIEW
This is the exploratory data analysis of the dataset using MySQL server.
Here I am just going to explore the data, to find trends, patterns and outliers

### Dataset Source
[Download here](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

### Tools
- MySQL Server

### Data Analysis

```sql

-- Here is the largest total layoffs 

SELECT MAX(total_laid_off), MAX(percentage_laid_off) FROM layoffs_static2;

-- Here are the details of the company which completely got laid off

SELECT * FROM layoffs_static2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;

-- To know the sum of total layoffs by company

SELECT company, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY company
ORDER BY 2 DESC;

-- To know the date from when the layoff started and the end date

SELECT MIN(`date`), MAX(`date`)
FROM layoffs_static2;

-- To know the sum of total layoffs by industry

SELECT industry, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY industry
ORDER BY 2 DESC;

-- To know the sum of total layoffs by country

SELECT country, SUM(total_laid_off)
FROM layoffs_static2
GROUP BY country
ORDER BY 2 DESC;

-- To know the sum of total layoffs by Year

SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_static2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;

-- To know the sum of total layoffs by Month

SELECT SUBSTRING(`date`, 1, 7) AS MONTH, SUM(total_laid_off)
FROM layoffs_static2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY MONTH
ORDER BY 1 ASC;

-- Rolling total of layoffs per month

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

-- Sum of total layoffs by company and year

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
