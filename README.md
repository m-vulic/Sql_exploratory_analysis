# Sql_exploratory_analysis
Exploratory_data_analysis

SELECT * FROM layoffs_backup3;

-- IM GONNA FOCUS ON total_laid_off, percentage_laid_off, `date` COLUMNS FOR EXPLORATORY ANALYSIS

-- WHAT IS MAX total_laid_off AND percentage_laid_off IN COMPANIES

SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_backup3;

-- WHICH COMPANIES LAID OFF ALL PEOPLE // 1 MEANS 100%

SELECT *
FROM layoffs_backup3
WHERE percentage_laid_off = 1;

-- TOP10 COMPANIES WHICH HAD THE LARGEST LAYOFFS THROUGHT THE YEARS

SELECT *
FROM layoffs_backup3
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC
LIMIT 10;

-- SUM OF LAID OFF BY THE COMPANY

SELECT company, SUM(total_laid_off) AS company_layoffs_qty
FROM layoffs_backup3
GROUP BY company
ORDER BY 2 DESC;

-- WHAT IS THE TIME PERIOD OF ANALYSIS IN LAYOFFS 

SELECT MIN(`date`), MAX(`date`)
FROM layoffs_backup3;

-- WHAT INDUSTRIES WERE HIT THE MOST DURING COVID19 LAYOFFS

SELECT 
    industry, SUM(total_laid_off) AS industry_layoffs_qty
FROM
    layoffs_backup3
GROUP BY industry
ORDER BY 2 DESC;

-- TOP5 COUNTRIES WHICH SUFFERED THE MOST LAYOFFS BECAUSE OF COVID

SELECT country, SUM(total_laid_off) AS country_layoffs_qty
FROM layoffs_backup3
GROUP BY country
ORDER BY 2 DESC
LIMIT 5;

-- TOTAL LAYOFFS PER YEAR

SELECT YEAR(`date`) AS 'YEAR', SUM(total_laid_off) AS total_layoffs
FROM layoffs_backup3
GROUP BY YEAR(`date`)
ORDER BY 1 ASC;

-- LAYOFFS PER MONTH EVOLUTION

SELECT SUBSTRING(`date`,1,7) AS 'MONTH', SUM(total_laid_off) AS total_layoffs
FROM layoffs_backup3
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY SUBSTRING(`date`,1,7)
ORDER BY 1 ASC;

-- COMPARING EVOLUTION OF LAYOFFS BY MONTH WITH TOTAL LAYOFFS - MAKING CTE FOR ROLLING EVOLUTION

WITH Evolution_Layoffs AS
(SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
FROM layoffs_backup3
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY SUBSTRING(`date`,1,7)
ORDER BY 1 ASC
)
SELECT `MONTH`, total_off,
sum(total_off) OVER(ORDER BY `MONTH`) AS evolution_total
FROM Evolution_Layoffs;

-- LAYOFFS EVOLUTION BY COMPANIES RANKING PER YEAR

WITH Company_Evolution (company, years, total_laid_off) AS
(SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_backup3
GROUP BY company, YEAR(`date`)
), Company_Evolution_Rank AS
(SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
FROM Company_Evolution
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Evolution_Rank
WHERE Ranking <= 5;

-- GREAT QUERY FOR VISUALISATION --
-- EVOLUTION OF LAYOFFS BY COMPANY RANKED TOP5 PER YEAR
