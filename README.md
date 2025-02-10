# SQL Data Cleaning

- Course: [Data Cleaning in MySQL by Alex The Analyst](https://www.youtube.com/watch?v=4UltKCnnnTA&t=14s)
- Source file: https://github.com/AlexTheAnalyst/MySQL-YouTube-Series/blob/main/layoffs.csv

# Tools
- MySQL Workbench

# Lessons
- Be ready that MySQL Workbench will upload only 400 rows insead of 2362. So, had to figure out how to create table and insert data into it using MySQL Workbench.
- Although during the course seprate column, which holds row id, is added, **the biggest takaway is to:
  - add row-id-column into the original file**
  - and after that start working with the working file, because there will be a mess, and it is much easier to restore/update data.
- Although it was SQL tutorial, in the future probably first cleaning (e.g. removing duplicates) is easier to do in Excel?
- Again, document while doing, because time-space doesn't helping remberging.

# Steps
'''sql
-- Standardizing data

-- SELECT distinct(trim(company)) FROM layoffs_working;
-- SELECT company, trim(company) FROM layoffs_working;
-- UPDATE layoffs_working SET company = trim(company);
'''
