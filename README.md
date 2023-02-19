# Analyzing-Defforestation-Data-Using-SQL

-- CREATING FORESTATION VIRTUAL TABLE


DROP VIEW IF EXISTS forestation; 
CREATE VIEW forestation AS
(SELECT f.country_code, f.country_name, f.year, f.forest_area_sqkm, l.total_area_sq_mi, r.region,
r.income_group,
l.total_area_sq_mi * 2.59 total_area_sqkm, ROUND(((f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100)::numeric,2)
percentage_forest_area
FROM forest_area f
JOIN land_area l
ON f.country_code=l.country_code AND f.year=l.year JOIN regions r
ON l.country_code=r.country_code);


---- TASK 1: GLOBAL SITUATION
WITH forest_area_1990 as (SELECT country_name,
year,
forest_area_sqkm FROM forestation
WHERE country_name='World' AND year=1990), forest_area_2016 as
(SELECT country_name,
year, forest_area_sqkm
FROM forestation
WHERE country_name='World' AND year=2016)
SELECT *
FROM forest_area_2016;


-- difference between 1990 and 2016
SELECT
(SELECT forest_area_sqkm
FROM forestation
WHERE country_name='World' AND year=1990)-
(SELECT forest_area_sqkm
FROM forestation
WHERE country_name='World' AND year=2016) as difference;


-- percentage change from 1990 to 2016
WITH forest_area_1990 as (SELECT country_name,
year, forest_area_sqkm FROM forestation
WHERE country_name='World' AND year=1990),
forest_area_2016 as
(SELECT country_name, year, forest_area_sqkm FROM forestation
WHERE country_name='World' AND year=2016),
diff AS
(SELECT f90.forest_area_sqkm a_1990,
f16.forest_area_sqkm a_2016, (f90.forest_area_sqkm - f16.forest_area_sqkm) diff, ((f90.forest_area_sqkm - f16.forest_area_sqkm)/ f90.forest_area_sqkm)*100 percentage_diff
FROM forest_area_1990 f90, forest_area_2016 f16)
SELECT a_1990,a_2016,diff, ROUND(percentage_diff::numeric,2) FROM diff;


--camparing area lost with entire country of that area
SELECT country_name,
ROUND(total_area_sqkm::numeric,2) total_area_sqkm
FROM forestation
WHERE (total_area_sqkm BETWEEN 1270000 AND 1350000) AND year=2016;


-- TASK 2: REGIONAL OUTLOOK
WITH forest_percentage_1990 as (Select region,
ROUND ((SUM (forest_area_sqkm)*100/
SUM(total_area_sqkm))::numeric,2) percent_1990 FROM forestation
WHERE year=1990
GROUP BY region
ORDER BY percent_1990 DESC),
forest_percentage_2016 as (Select region,
ROUND ((SUM (forest_area_sqkm)*100/
SUM(total_area_sqkm))::numeric,2) percent_2016 FROM forestation
WHERE year=2016
GROUP BY region
ORDER BY percent_2016 DESC),
joined_1990_2016 as

(Select fp1990.region, fp1990.percent_1990,
fp2016.percent_2016 FROM forest_percentage_1990 fp1990
JOIN forest_percentage_2016 fp2016 ON fp1990.region=fp2016.region)
Select *
FROM joined_1990_2016;


---TASK 3- COUNTRY-LEVEL DETAIL
--Success story
WITH Forest_area_1990 AS (SELECT country_name,
year, forest_area_sqkm, region
FROM forestation
WHERE year=1990 AND forest_area_sqkm IS NOT NULL),
Forest_area_2016 AS
(SELECT country_name,year,forest_area_sqkm,region FROM forestation
WHERE year=2016 AND forest_area_sqkm IS NOT NULL),
Difference_forest_area AS
(SELECT FA90.country_name,
FA16.region, FA90.year year_1990,
FA16.year year_2016 ,
FA90.forest_area_sqkm AREA_1990, FA16.forest_area_sqkm AREA_2016, (FA16.forest_area_sqkm-FA90.forest_area_sqkm) Diff, ((FA16.forest_area_sqkm-
FA90.forest_area_sqkm)*100/FA90.forest_area_sqkm) per_decrease_forest_area FROM forest_area_1990 FA90
JOIN forest_area_2016 FA16
ON FA90.country_name=FA16.country_name
ORDER BY Diff desc)
SELECT df.country_name,
df.region,ROUND(df.diff::numeric,2) as Change_in_forest_area, ROUND(df.per_decrease_forest_area::numeric,2) per_decrease_forest_area
FROM Difference_forest_area df
ORDER BY per_decrease_forest_area DESC LIMIT 6;


--- top 5 countries with largest amount decrease in forest area and difference in forest area for each
WITH Forest_area_1990 AS (SELECT country_name,
year, forest_area_sqkm, region
FROM forestation
WHERE year=1990 AND forest_area_sqkm IS NOT NULL),
Forest_area_2016 AS
(SELECT country_name,year,forest_area_sqkm,region FROM forestation
WHERE year=2016 AND forest_area_sqkm IS NOT NULL),
Difference_forest_area AS
(SELECT FA90.country_name,
FA16.region, FA90.year year_1990,
FA16.year year_2016 ,
FA90.forest_area_sqkm AREA_1990, FA16.forest_area_sqkm AREA_2016, (FA90.forest_area_sqkm-FA16.forest_area_sqkm) Diff,
FROM forest_area_1990 FA90
JOIN forest_area_2016 FA16
ON FA90.country_name=FA16.country_name ORDER BY Diff )
SELECT df.country_name, df.region,df.area_1990,df.area_2016,ROUND(df.diff::numeric,2) as
Change_in_forest_area, FROM Difference_forest_area df LIMIT 6;


---5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each
 
 
 WITH Forest_area_1990 AS
 (SELECT country_name,year,
 forest_area_sqkm,region,
 total_area_sqkm
 FROM forestation
 WHERE year=1990 AND forest_area_sqkm IS NOT NULL),
 Forest_area_2016 AS
 (SELECT country_name,year,
 FROM forestation
forest_area_sqkm,region,
 total_area_sqkm
  WHERE year=2016 AND forest_area_sqkm IS NOT NULL),
 Difference_forest_area AS
 (SELECT FA90.country_name,
 FA16.region,
 FA90.year year_1990,
 FA16.year year_2016 ,
 FA90.total_area_sqkm totalarea1990,
 FA16.total_area_sqkm totalarea2016,
 FA90.forest_area_sqkm AREA_1990,
 FA16.forest_area_sqkm AREA_2016,
 (FA16.forest_area_sqkm-FA90.forest_area_sqkm) Diff,
 ((FA16.forest_area_sqkm-FA90.forest_area_sqkm)*100/
 FA90.forest_area_sqkm) per_decrease_forest_area
 FROM forest_area_1990 FA90
 JOIN forest_area_2016 FA16
 ON FA90.country_name=FA16.country_name
 ORDER BY Diff)
 SELECT df.country_name,
 df.region,
 ROUND(df.per_decrease_forest_area::numeric,2) per_dec_forest_area
 FROM Difference_forest_area df
 ORDER BY per_dec_forest_area
 LIMIT 5;
 

 --- If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016
 
 
 WITH forest_percentage AS
 (SELECT country_name,percentage_forest_area
 FROM forestation
 WHERE year=2016 AND percentage_forest_area IS NOT NULL),
 T1 AS
 (SELECT fp.country_name, fp.percentage_forest_area,
 CASE
 WHEN fp.percentage_forest_area>=75 THEN 'Q4'
 WHEN fp.percentage_forest_area>=50 THEN 'Q3'
 FROM forest_percentage fp
WHEN fp.percentage_forest_area>=25 THEN 'Q2'
 ELSE 'Q1'
 END AS Quartiles
  WHERE percentage_forest_area IS NOT NULL AND country_name!='World')
 SELECT Quartiles, count(*)
 FROM T1
 GROUP BY 1
 ORDER BY 1;
 
 
--List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.
WITH forest_percentage AS
(SELECT country_name,
T1 AS
percentage_forest_area,
region FROM forestation
WHERE year=2016 AND percentage_forest_area IS NOT NULL),
(SELECT fp.country_name, fp.region,
fp.percentage_forest_area, CASE

WHEN fp.percentage_forest_area>=75 THEN 'Q4' WHEN fp.percentage_forest_area>=50 THEN 'Q3' WHEN fp.percentage_forest_area>=25 THEN 'Q2' ELSE 'Q1'
END AS Quartiles FROM forest_percentage fp
WHERE percentage_forest_area IS NOT NULL AND country_name!='World')
SELECT Country_name,region,percentage_forest_area FROM T1
WHERE Quartiles='Q4'
ORDER BY 3 DESC;
