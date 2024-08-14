# SQL Project: Data Cleaning
### PROJECT OVERVIEW
This is a project based on the cleaning of a dataset with the use of SQL server specifically MySQL

### Dataset Source
[Download here](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

### Tools
- MySQL Server

### Data Analysis
``` sql

-- Removal of duplicates from the dataset

SELECT * FROM layoffs;

 -- I created another table called layoffs_static so as to be able to clean the data while keeping the raw data safe in case of changes.

CREATE TABLE layoffs_static;

-- I inserted the data from the layoffs table into the layoffs_static table

INSERT INTO layoffs_static
SELECT * FROM layoffs;

-- I checked for duplicates in the dataset using this method

SELECT *,
ROW_NUMBER()
OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions)
AS row_num
FROM 
layoffs_static;

-- I created a CTE function to easily check for duplicates, if the row_num is greater than 1, those are the duplicates 

WITH duplicate_cte AS
(SELECT *,
ROW_NUMBER()
OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions)
AS row_num
FROM 
layoffs_static)

SELECT * FROM duplicate_cte
WHERE row_num > 1;

-- To delete the duplicates, I created another table layoffs_static2 so as to be extra careful, then I inserted a new column 'row_num' with the data type INT, then I inserted the layoffs_static with duplicates into the layoffs_static2

CREATE TABLE `layoffs_static2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  row_num INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO layoffs_static2
SELECT *,
ROW_NUMBER()
OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions )
AS row_num
FROM 
layoffs_static;

-- I deleted the duplicates from the layoffs_static2

DELETE FROM layoffs_static2
WHERE row_num > 1; 

This is the table with no duplicates

SELECT * FROM layoffs_static2
WHERE row_num > 1;

-- Standardization of the dataset

-- I removed any white spaces from the company row using the trim function and I updated the company row to a cleaner version

SELECT company, TRIM(company) FROM layoffs_static2;

UPDATE layoffs_static2
SET company = TRIM(company);

-- I checked to see the distinct industry and I noticed Crypto was written in two different ways

SELECT DISTINCT industry
FROM layoffs_static2 ;

-- I standardized the name into a single name

UPDATE layoffs_statitic2
SET industry = 'Crypto'
WHERE industry LIKE Crypto% ;

-- I noticed there was a full stop at the end of United States which was different from the one without full stop and it could give inconsistencies in visualizing the data.

SELECT DISTINCT country FROM layoffs_static2;

-- There are two methods I used in doing this:

-- I used the trim, then the trailing function to remove any full stop behind the country row

UPDATE layoffs_static2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%'
ORDER BY country;

-- I updated the table and set the country into a standardized form

UPDATE layoffs_static2
SET country = 'United States'
WHERE country LIKE 'United States%';

-- The date column was in text format, so I updated the date column to a date format using the str_to_date function

SELECT `date` ,
STR_TO_DATE(`date`, '%m/%d/%Y' )
FROM layoffs_static2;

UPDATE layoffs_static2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- The date column still had the text data type, so i altered the table and modified the date column and changed the data type into date

ALTER TABLE layoffs_static2
MODIFY COLUMN `date` DATE;

-- Removal of Null and blank values not necessary to the dataset

SELECT * FROM layoffs_static2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

SELECT * FROM layoffs_static2
where industry IS NULL OR 
industry = '';

SELECT t1.industry, t2.industry FROM layoffs_static2 t1
JOIN layoffs_static2 t2
ON t1.company = t2.company 
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs_static2 t1
JOIN layoffs_static2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL; 
 
UPDATE layoffs_static2
SET industry = NULL
WHERE industry = '';
 
 SELECT * FROM layoffs_static2;
 
 ALTER TABLE layoffs_static2
 DROP COLUMN row_num

```

