# SQL Data Cleaning

- Course: [Data Cleaning in MySQL by Alex The Analyst](https://www.youtube.com/watch?v=4UltKCnnnTA&t=14s)
- Source file: https://github.com/AlexTheAnalyst/MySQL-YouTube-Series/blob/main/layoffs.csv

# Tools
- MySQL Workbench

# Lessons
- Be ready that MySQL Workbench will upload only 400 rows insead of 2362. So, had to figure out how to create table and insert data into it using MySQL Workbench.
- Although during the course seprate column is added, which holds row id, the biggest takaway is:
  - **add row-id-column into the original file**
  - and after that start working with the working file, because there will be a mess, and it is much easier to restore/update data.
- Although it was SQL tutorial, in the future probably first cleaning (e.g. removing duplicates) is easier to do in Excel?
- deleting
  - column: ALTER TABLE table_name DROP COLUMN column_name;
  - row: DELETE FROM table_name WHERE condition;
  - table: DROP TABLE table_name;
- ROW_NUMBER
  - SELECT  ROW_NUMBER() OVER (ORDER column_name/PARTICION BY column_name) AS row_num, column1, column2 FROM table_name;
  - Copilot: ROW_NUMBER is a window function in SQL that assigns a unique sequential integer to rows within a result set, starting at 1 for the first row. It can be very useful when you need to assign unique IDs to rows or when you need to sort and paginate results.
- PARTITION BY
  - SELECT department, employee_name, salary, AVG(salary) OVER (PARTITION BY department) AS avg_salary_per_dept, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_within_dept FROM 
 employees;
  - AVG(salary) OVER (PARTITION BY department) calculates the average salary within each department.
  - ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) assigns a rank to each employee within their department based on their salary, with the highest salary getting rank 1.
- Again, document while doing, because time-space doesn't help rembembering.

# Steps


```sql
--Removing spaces from Company's name and updating them.

SELECT distinct(trim(company)) FROM layoffs_working;
SELECT company, trim(company) FROM layoffs_working;
UPDATE layoffs_working SET company = trim(company);

```

```sql
-- Correcting und updating the Crypto-industry rows.

SELECT distinct industry from layoffs_working order by 1;
UPDATE layoffs_working SET industry = "Crypto" WHERE industry like 'Crypto%';

```

```sql
-- Correcting spelling and updating the rows where the country is 'United States'.

SELECT distinct(country) FROM layoffs_working WHERE country LIKE "United States%" order by 1;
SELECT country, trim(trailing '.' from country) FROM layoffs_working WHERE country LIKE "United States%";
UPDATE layoffs_working SET country = trim(trailing '.' from country) WHERE country like 'United States%';
-- UPDATE layoffs_working SET country = trim(country);
```


```sql
-- Finding duplicates: let's choose columns and find where the data is the same in both rows and there is more than 1 row

SELECT company, total_laid_off, percentage_laid_off, industry, location, COUNT(*) as count FROM layoffs_working GROUP BY company, total_laid_off, percentage_laid_off, industry, location HAVING COUNT(*) >1;

![image](https://github.com/user-attachments/assets/b69e2db8-12f5-4544-a402-33eb5d6055cd)
![image](https://github.com/user-attachments/assets/3f5df42f-6671-4966-9bd6-aa4aa56150bf)


```

```sql
-- adding row_id column into the table and incremented
```
```sql

-- Finding rows there is no data how many people were fired.

SELECT * FROM layoffs_working WHERE total_laid_off IS NULL;
SELECT * FROM layoffs_working WHERE total_laid_off = "NULL" AND percentage_laid_off = "NULL";
DELETE FROM layoffs_working WHERE total_laid_off= "NULL" AND percentage_laid_off = "NULL";
SELECT * FROM layoffs_working WHERE total_laid_off IS NULL or total_laid_off ="";
```

```sql
-- Finding rows where is no data about the industry.

SELECT * from layoffs_working WHERE industry = "NULL";
SELECT * FROM layoffs_working WHERE company like "Bally's%";
UPDATE layoffs_working SET industry = "Sports betting" where company like "Bally%"


```


```sql

-- Populating data - looking for industry from the same table if the company is mentioned several times


SELECT * FROM layoffs_working wHERE company = 'Airbnb';

SELECT 
    *
FROM
    layoffs_working AS t1
        JOIN
    layoffs_working AS t2 ON t1.company = t2.company
        AND t1.location = t2.location
        WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;
        
        
UPDATE layoffs_working AS t1
JOIN layoffs_working AS t2 
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;


```


```sql
-- Date has always been difficult to handle. In this tutorial date was left as text field during creation of the table

SELECT company, `date`, total_laid_off, percentage_laid_off, industry, location, COUNT(*) as count FROM layoffs_working GROUP BY company, total_laid_off, percentage_laid_off, industry, location HAVING COUNT(*) > 1;
SELECT company, `date`, total_laid_off, percentage_laid_off, industry, location, COUNT(*) as count FROM layoffs_working GROUP BY company, `date`, total_laid_off, percentage_laid_off, industry, location HAVING COUNT(*) > 1;

SELECT layoffs_working.company, layoffs_working.date AS date_working, layoffs.date AS date_layoff, layoffs_working.total_laid_off, layoffs_working.percentage_laid_off, layoffs_working.industry, layoffs_working.location, COUNT(*) as count FROM layoffs_working JOIN layoffs ON layoffs_working.company = layoffs.company AND layoffs_working.total_laid_off = layoffs.total_laid_off AND layoffs_working.percentage_laid_off = layoffs.percentage_laid_off AND layoffs.industry = layoffs_working.industry AND layoffs.location = layoffs_working.location GROUP BY layoffs_working.company, layoffs_working.date, layoffs.date, layoffs_working.total_laid_off, layoffs_working.percentage_laid_off, layoffs_working.industry, layoffs_working.location HAVING COUNT(*) > 1;


-- And made new column as type DATE
ALTER TABLE layoffs_working DROP COLUMN date_converted;
ALTER TABLE layoffs_working ADD COLUMN date_converted DATE:

SELECT `date`
FROM layoffs_working
WHERE STR_TO_DATE(`date`, '%d/%m/%Y') IS NULL;
SELECT `date`, id FROM layoffs_working WHERE STR_TO_DATE(TRIM(`date`), '%m/%d/%Y') IS NULL;


UPDATE layoffs_working
SET `date` = DATE_FORMAT(STR_TO_DATE(TRIM(`date`), '%m/%d/%Y'), '%Y-%m-%d')
WHERE STR_TO_DATE(TRIM(`date`), '%m/%d/%Y') IS NOT NULL;
```

```sql
-- As column date is still text-format; column date_converted is in date-format

UPDATE layoffs_working 
 SET date_converted = STR_TO_DATE(`date`, '%Y-%m-%d');
SELECT * FROM layoffs_working;
-- NULL vs "NULL"
-- NULL means lack of info or no info. "NULL" is a strng and usually not recommended.
SELECT * FROM layoffs_working WHERE  `total_laid_off` = 'Null' OR `percentage_laid_off` = 'Null' or "funds_raised_millions" = "Null";
UPDATE layoffs_working SET `date` = DATE_FORMAT(STR_TO_DATE(`date`, '%d/%m/%Y'), '%d/%m/%Y');
UPDATE layoffs_working SET `total_laid_off` = NULL, `percentage_laid_off` = NULL, `funds_raised_millions` = NULL WHERE `total_laid_off` = 'Null' OR `percentage_laid_off` = 'Null' OR `funds_raised_millions` = 'Null';
SELECT * FROM layoffs_working WHERE `total_laid_off` = 'Null' OR `percentage_laid_off` = 'Null' OR `funds_raised_millions` = 'Null';
```

```sql
--added column and then changing it

ALTER TABLE layoffs_working drop column `date`;
ALTER TABLE layoffs_working CHANGE COLUMN `  ` `del1` INT;

```
