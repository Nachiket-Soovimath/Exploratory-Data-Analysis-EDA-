# Exploratory-Data-Analysis-EDA-
 <h1>Exploratory Data Analysis (EDA) - Layoffs Data</h1>
        <p>This document provides a detailed analysis of the layoffs dataset using SQL queries. The goal is to gain insights about the layoffs across different industries, countries, companies, and years.</p>
    </header>

   
        <h2>1. Initial Data Overview</h2>
        <p>The first query retrieves all the columns from the <code>layoffs_staging2</code> table, which gives us a snapshot of the data in its entirety.</p>
        <pre><code>
SELECT * FROM layoffs_staging2;
        </code></pre>

        <h3>2. Maximum Number of Layoffs and Percentage</h3>
        <p>This query retrieves the maximum number of layoffs and the maximum percentage of layoffs observed in the dataset. It helps to identify extreme values in the dataset.</p>
        <pre><code>
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
        </code></pre>

        <h3>3. Companies with 100% Layoffs (Fully Laid Off)</h3>
        <p>This query finds companies where the percentage of layoffs is 100% (i.e., all employees were laid off) and orders them by the amount of funds raised in descending order.</p>
        <pre><code>
SELECT * 
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
        </code></pre>

        <h3>4. Total Layoffs by Company</h3>
        <p>Here, we aggregate the total number of layoffs by each company and order them in descending order. This allows us to understand which companies experienced the highest layoffs.</p>
        <pre><code>
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
        </code></pre>

        <h3>5. Date Range of Layoffs</h3>
        <p>This query provides the minimum and maximum date from the <code>date</code> column, helping to identify the period over which layoffs have occurred in the dataset.</p>
        <pre><code>
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;
        </code></pre>

        <h3>6. Total Layoffs by Industry</h3>
        <p>This query aggregates layoffs by industry, ordering them in descending order. It helps to identify which industries have been most affected by layoffs.</p>
        <pre><code>
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
        </code></pre>

        <h3>7. Total Layoffs by Country</h3>
        <p>This query aggregates the total layoffs by country, which can reveal geographical trends in layoffs.</p>
        <pre><code>
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
        </code></pre>

        <h3>8. Total Layoffs by Year</h3>
        <p>Here, we group the data by year and calculate the total layoffs for each year. This allows us to examine the trend of layoffs over time.</p>
        <pre><code>
SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;
        </code></pre>

        <h3>9. Total Layoffs by Stage</h3>
        <p>This query aggregates layoffs by the "stage" column (such as pre-seed, seed, series A, etc.) and helps identify the relationship between layoffs and the company's growth stage.</p>
        <pre><code>
SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 1 DESC;
        </code></pre>
 

        <h2>10. Rolling Sum of Layoffs</h2>
        <p>This section explores the rolling sum of layoffs by month. We first create a temporary table called <code>Rolling_Total</code> to group layoffs by month and calculate the total layoffs for each month. We then compute a cumulative sum (rolling total) of layoffs over time.</p>
        <pre><code>
WITH Rolling_Total AS
(
    SELECT SUBSTRING(`date`, 1, 7) AS `MONTH`, SUM(total_laid_off) AS total_off
    FROM layoffs_staging2
    WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
    GROUP BY `MONTH`
    ORDER BY 1 ASC
)
SELECT `MONTH`, total_off, SUM(total_off) OVER(ORDER BY `MONTH`) AS Rolling
FROM Rolling_Total;
        </code></pre>

        <h3>11. Total Layoffs by Company and Year</h3>
        <p>This query aggregates total layoffs by company and year, which allows us to observe how layoffs have changed over time for each company.</p>
        <pre><code>
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY 3 DESC;
        </code></pre>

        <h3>12. Ranking of Top 5 Companies with Most Layoffs per Year</h3>
        <p>This query first calculates the total layoffs per company and year. Then, it ranks the companies by the number of layoffs in descending order for each year, and selects the top 5 companies with the most layoffs per year.</p>
        <pre><code>
WITH company_year(company, years, total_laid_off) AS
(
    SELECT company, YEAR(`date`), SUM(total_laid_off)
    FROM layoffs_staging2
    GROUP BY company, YEAR(`date`)
),
Company_year_Rank AS
(
    SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
    FROM company_year
    WHERE years IS NOT NULL
)
SELECT * 
FROM Company_year_Rank
WHERE Ranking <= 5;
        </code></pre>
    <h2>Conclusion</h2>
        <p>The above SQL queries provide a comprehensive view of the layoffs dataset, offering insights into the companies, industries, countries, and periods most affected by layoffs. The rolling sum helps in identifying trends over time, while ranking companies by layoffs per year gives a clearer view of which companies experienced the most significant layoffs.</p>
