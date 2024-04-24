# Data-Cleaning-using-SQL
Examining the data to identify common issues such as missing values, duplicates, incorrect data types, and outliers. This may involve running queries to select and inspect subsets of the data.
-- Data Cleaning

Select * from layoffs;

-- 1. Remove Duplicates
-- 2. Standardize the Data
-- 3. Decide on what to do with Null Values or blank values
-- 4. Remove unwanted columns/rows but double check for its needs.

-- Create a duplicate table in order to keep original dataset
CREATE TABLE layoffs_staging LIKE layoffs;
SELECT * FROM layoffs_staging;

-- Insert data from the layoffs table into the layoffs_staging table
INSERT layoffs_staging SELECT * FROM layoffs;

SELECT * FROM layoffs_staging;

-- Since no unique ID to identify duplicates, add row numbers as a unique identifier at the end to get duplicates

SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off,
'date', stage, country, funds_raised_millions) AS row_num FROM layoffs_staging; 

WITH Duplicate_cte AS 
( SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off,
`date`, stage, country, funds_raised_millions) AS row_num FROM layoffs_staging)
SELECT * FROM Duplicate_cte WHERE row_num > 1;

-- Create Another table to insert the row number column 
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- Insert data from layoffs_staging table
INSERT INTO layoffs_staging2
SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off,
'date', stage, country, funds_raised_millions) AS row_num FROM layoffs_staging; 

SELECT * FROM layoffs_staging2;

-- After identifying duplicate rows, the rows should be deleted
DELETE FROM layoffs_staging2 WHERE row_num > 1;

SELECT * FROM layoffs_staging2;

-- Standardizing Data (Finding issues with individual columns in your data and fixing it)
SELECT company FROM layoffs_staging2;
SELECT company, trim(company) FROM layoffs_staging2;
UPDATE layoffs_staging2 SET company = trim(company);

SELECT DISTINCT location FROM layoffs_staging2;

SELECT DISTINCT industry FROM layoffs_staging2;
SELECT * FROM layoffs_staging2 WHERE industry LIKE 'crypto%';

UPDATE layoffs_staging2 SET industry = 'crpto' WHERE industry LIKE 'crypto%';

SELECT * FROM layoffs_staging2;

SELECT DISTINCT country FROM layoffs_staging2 ORDER BY 1;

SELECT DISTINCT country, trim(TRAILING '.' FROM country) FROM layoffs_staging2 ORDER BY 1;
UPDATE layoffs_staging2
SET country = trim(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

SELECT `date`, str_to_date(`date`, '%m/%d/%Y') FROM layoffs_staging2;

UPDATE layoffs_staging2 SET `date` = str_to_date(`date`, '%m/%d/%Y');

SELECT `date` FROM layoffs_staging2;

ALTER TABLE layoffs_staging2 MODIFY COLUMN `date` DATE;

-- Columns with linked values but both missing 
SELECT * FROM layoffs_staging2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

SELECT * FROM layoffs_staging2
WHERE industry IS NULL OR industry = '';

-- Inspect the table for the individual company names
SELECT * FROM layoffs_staging2
WHERE company = 'Airbnb';

-- Update blank columns in industry by  self joining
SELECT * FROM layoffs_staging2 t1 JOIN layoffs_staging2 t2
ON t1.company = t2.company WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

-- Set Blank Values to Null before the Update function for the column
Update layoffs_staging2
Set industry = Null Where industry = '';

SELECT t1.industry, t2.industry FROM layoffs_staging2 t1 JOIN layoffs_staging2 t2
ON t1.company = t2.company WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2 
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL and t2.industry IS NOT NULL;

SELECT * FROM layoffs_staging2;

-- Dropping the column row number created at the beginning
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
