# Schema

```sql
CREATE DATABASE `Data Cleaning Project`;
```
```sql
select * 
from layoffs;
```

## 1. Remove duplicates
```sql
create table layoffs_staging -- I created thisto make changes to the data without altering the raw data
like layoffs;
```
```sql
select * 
from layoffs_staging;
```
```sql
insert layoffs_staging
select * 
from layoffs;
```
```sql
select *,
row_number() over(
partition by company, industry, total_laid_off, percentage_laid_off, 'date') as Row_num
from layoffs_staging;
```
```sql
with duplicate_cte as
(
select *,
row_number() over(
partition by company, industry, total_laid_off, percentage_laid_off, 'date',
stage, country, funds_raised_millions) as Row_num
from layoffs_staging
) 
select *
from duplicate_cte
where row_num > 1;
```
```sql
CREATE TABLE `layoffs_staging3` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
```sql
insert into layoffs_staging3
select *,
row_number() over(
partition by company, industry, total_laid_off, percentage_laid_off, 'date',
stage, country, funds_raised_millions) as Row_num
from layoffs_staging;
```
```sql
select *
from layoffs_staging3
where row_num = 1;
```
```sql
delete
from layoffs_staging3
where row_num > 1;
```
## 2. Standardize the Data (spellings)
```sql
-- company
select company, trim(company)
from layoffs_staging3;
```
```sql
update layoffs_staging3
set company = trim(company);
```
```sql
-- industry
select distinct industry
from layoffs_staging3
order by 1;
```
```sql
select * 
from layoffs_staging3
where industry like 'crypto%';
```
```sql
update layoffs_staging3
set industry =  'Crypto'
where industry like 'crypto%';
```
```sql
-- Country
select distinct country, trim(trailing '.' from country) 
from layoffs_staging3
order by 1;
```
```sql
update  layoffs_staging3
set country = trim(trailing '.' from country)
where country like 'united States%';
```
```sql
-- Date
-- change date format to date.
select `date` -- ,
-- str_to_date(`date`, '%m/%d/%Y')
from layoffs_staging3;
```
```sql  
update layoffs_staging3
set `date` = str_to_date(`date`, '%m/%d/%Y');
```
```sql
-- change to date format
alter table layoffs_staging3
modify column `date` DATE;
```
## 3. Null Values or Blank Values
```sql
select *
from layoffs_staging3
where total_laid_off is null
and percentage_laid_off is null;
```
```sql
select *
from layoffs_staging3
where industry is null
or industry = '';
```
```sql
select *
from layoffs_staging3
where company like 'bally%';
```
```sql
-- we use the filled duplicate to surfice the empty one
select *
from layoffs_staging3 t1
join layoffs_staging3 t2
	on t1.company = t2.company
where (t1.industry is null or t1.industry = '')
and t2.industry is not null;
```
```sql
update layoffs_staging3
set industry = null
where industry = '';
```
```sql
UPDATE layoffs_staging3 t1
join layoffs_staging3 t2
	on t1.company = t2.company
set t1.industry = t2.industry
where t1.industry is null
and t2.industry is not null;
```
```sql
select *
from layoffs_staging3;
```

## 4. Remove Any Columns or Rows, if (irrelevant)
```sql
delete
from layoffs_staging3
where total_laid_off is null
and percentage_laid_off is null;
```
```sql
-- Drop the row_num column
alter table layoffs_staging3
drop column row_num;
```
