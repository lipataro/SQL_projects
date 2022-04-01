# Project 1 - SQL

### Import data into SQL client and change names

- Download following csv file: [https://www.data.gv.at/katalog/dataset/bad388c1-e13f-484d-ba51-331a79537f5f](https://www.data.gv.at/katalog/dataset/bad388c1-e13f-484d-ba51-331a79537f5f)
- Upload csv file on your database using DBeaver import tool (find out how).
- make sure you pay attention to delimiter (, or ; or |)
- Uploaded table should have 4 columns
- Column names are ***Messort, Parameter, Zeitpunkt, HMW***
- Change the table name to ***opendata_meteo***
- Change the column names:
    - ***Messort -> measurement_location***
    - ***Zeitpunkt -> measurement_time***

```sql
ALTER TABLE NewTable RENAME opendata_meteo;
ALTER TABLE opendata_meteo CHANGE Messort measurement_location varchar(1024);
ALTER TABLE opendata_meteo CHANGE Zeitpunkt measurement_time varchar(64);
SELECT * FROM opendata_meteo LIMIT 5;
```

| measurement_location | Parameter | measurement_time | HMW |
| --- | --- | --- | --- |
| Salzburg Flughafen | Lufttemperatur [GradC] | 04.02.2022 00:30 | 3,7 |
| Salzburg Flughafen | Lufttemperatur [GradC] | 04.02.2022 01:00 | 4,0 |
| Salzburg Flughafen | Lufttemperatur [GradC] | 04.02.2022 01:30 | 3,4 |
| Salzburg Flughafen | Lufttemperatur [GradC] | 04.02.2022 02:00 | 3,1 |
| Salzburg Flughafen | Lufttemperatur [GradC] | 04.02.2022 02:30 | 3,3 |

### Develop the columns into a new table

- Create a new table called ***opendata_meteo_wide*** by querying uploaded table in which you find all distinct values of **Parameter** and per distinct parameter you create a new column.

```sql
CREATE TABLE opendata_meteo_wide AS SELECT * FROM opendata_meteo ;
SELECT DISTINCT Parameter FROM opendata_meteo_wide;
```

| Parameter |
| --- |
| Lufttemperatur [GradC] |
| rel. Luftfeuchte [%] |
| Windgeschwindigkeit [m/s] |
| Windrichtung [Grad] |
| Windspitze [m/s] |
| Luftdruck [hPa] |
| Sonnenscheindauer [min] |

```sql
ALTER TABLE opendata_meteo_wide 
	ADD COLUMN `Lufttemperatur [GradC]` varchar(64),
	ADD COLUMN `rel. Luftfeuchte [%]` varchar(64), 
	ADD COLUMN `Windgeschwindigkeit [m/s]` varchar(64),
	ADD COLUMN `Windrichtung  [Grad]` varchar(64),
	ADD COLUMN `Windspitze [m/s]` varchar(64), 
	ADD COLUMN `Luftdruck [hPa]` varchar(64), 
	ADD COLUMN `Sonnenscheindauer [min]` varchar(64);

UPDATE opendata_meteo_wide SET `Lufttemperatur [GradC]` = HMW WHERE Parameter = 'Lufttemperatur [GradC]';
UPDATE opendata_meteo_wide SET `rel. Luftfeuchte [%]` = HMW WHERE Parameter = 'rel. Luftfeuchte [%]';
UPDATE opendata_meteo_wide SET `Windgeschwindigkeit [m/s]` = HMW WHERE Parameter = 'Windgeschwindigkeit [m/s]';
UPDATE opendata_meteo_wide SET `Windrichtung  [Grad]` = HMW WHERE Parameter = 'Windrichtung  [Grad]';
UPDATE opendata_meteo_wide SET `Windspitze [m/s]` = HMW WHERE Parameter = 'Windspitze [m/s]';
UPDATE opendata_meteo_wide SET `Luftdruck [hPa]` = HMW WHERE Parameter = 'Luftdruck [hPa]';
UPDATE opendata_meteo_wide SET `Sonnenscheindauer [min]` = HMW WHERE Parameter = 'Sonnenscheindauer [min]';

ALTER TABLE opendata_meteo_wide DROP COLUMN HMW;
ALTER TABLE opendata_meteo_wide DROP COLUMN Parameter;
```

- Change column ***measurement_time***’s format to date.
    
    Before changing the table, we could test if our approach using **STR_TO_DATE** actually works:
    

```sql
SELECT measurement_time, STR_TO_DATE(measurement_time , "%d.%m.%Y  %H:%i") 
FROM opendata_meteo_wide;
```

and then update the table:

```sql
UPDATE opendata_meteo_wide 
SET measurement_time = STR_TO_DATE(measurement_time , "%d.%m.%Y  %H:%i" );

ALTER TABLE opendata_meteo_wide 
CHANGE measurement_time  measurement_time datetime;

SELECT * 
FROM opendata_meteo_wide LIMIT 5;
```

| measurement_location | measurement_time | Lufttemperatur [GradC] | rel. Luftfeuchte [%] | Windgeschwindigkeit [m/s] | Windrichtung  [Grad] | Windspitze [m/s] | Luftdruck [hPa] | Sonnenscheindauer [min] |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Salzburg Flughafen | ‣ | 3,7 | NULL | NULL | NULL | NULL | NULL | NULL |
| Salzburg Flughafen | ‣ | 4,0 | NULL | NULL | NULL | NULL | NULL | NULL |
| Salzburg Flughafen | ‣ | 3,4 | NULL | NULL | NULL | NULL | NULL | NULL |
| Salzburg Flughafen | ‣ | 3,1 | NULL | NULL | NULL | NULL | NULL | NULL |
| Salzburg Flughafen | ‣ | 3,3 | NULL | NULL | NULL | NULL | NULL | NULL |

### Analysing the data

- Have a look at the quality of the data
- Count how many missing values per columns exist. How many rows does the table have?
- What is the portion of missing values per column?
- Are there any other issues with the data quality?

```sql
SELECT
  COUNT(*) as `Total rows`,
  SUM(CASE WHEN `Lufttemperatur [GradC]` IS NULL THEN 1 ELSE 0 END) as `Missing Lufttemperatur`,
  SUM(CASE WHEN `rel. Luftfeuchte [%]` IS NULL THEN 1 ELSE 0 END) as `Missing rel. Luftfeuchte`, 
  SUM(CASE WHEN `Windgeschwindigkeit [m/s]` IS NULL THEN 1 ELSE 0 END) as `Missing Windgeschwindigkeit`,
  SUM(CASE WHEN `Windrichtung  [Grad]` IS NULL THEN 1 ELSE 0 END) as `Missing Windrichtung`,
  SUM(CASE WHEN `Windspitze [m/s]` IS NULL THEN 1 ELSE 0 END) as `Missing Windspitze`, 
  SUM(CASE WHEN `Luftdruck [hPa]` IS NULL THEN 1 ELSE 0 END) as `Missing Luftdruck`,
  SUM(CASE WHEN `Sonnenscheindauer [min]` IS NULL THEN 1 ELSE 0 END) as `Missing Sonnenscheindauer`
FROM opendata_meteo_wide;
```

| Total rows | NULL Lufttemperatur | NULL rel. Luftfeuchte | NULL Windgeschwindigkeit | NULL Windrichtung | NULL Windspitze | NULL Luftdruck | NULL Sonnenscheindauer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2015 | 1631 | 1631 | 1631 | 1631 | 1632 | 1967 | 1967 |

As the column Parameter was spread into different columns, the new table **opendata_meteo_wide** has a considerable portion of NULL values. However the original data contains missing measurements denoted by "**?**". Let's count how many *missing values* ("?") there are per column (or Parameter). 

```sql
SELECT
  COUNT(*) as `Total rows`,
  SUM(CASE WHEN `Lufttemperatur [GradC]`="?" THEN 1 ELSE 0 END) as `Missing Lufttemperatur`,
  SUM(CASE WHEN `rel. Luftfeuchte [%]`="?" THEN 1 ELSE 0 END) as `Missing rel. Luftfeuchte`, 
  SUM(CASE WHEN `Windgeschwindigkeit [m/s]`="?" THEN 1 ELSE 0 END) as `Missing Windgeschwindigkeit`,
  SUM(CASE WHEN `Windrichtung  [Grad]`="?" THEN 1 ELSE 0 END) as `Missing Windrichtung`,
  SUM(CASE WHEN `Windspitze [m/s]`="?" THEN 1 ELSE 0 END) as `Missing Windspitze`, 
  SUM(CASE WHEN `Luftdruck [hPa]`="?" THEN 1 ELSE 0 END) as `Missing Luftdruck`,
  SUM(CASE WHEN `Sonnenscheindauer [min]`="?" THEN 1 ELSE 0 END) as `Missing Sonnenscheindauer`
FROM opendata_meteo_wide;
```

| Total rows | Missing Lufttemperatur | Missing rel. Luftfeuchte | Missing Windgeschwindigkeit | Missing Windrichtung | Missing Windspitze | Missing Luftdruck | Missing Sonnenscheindauer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2015 | 240 | 240 | 240 | 240 | 239 | 30 | 30 |

And now the portion of missing values per column. 

```sql
SELECT
  SUM(CASE WHEN `Lufttemperatur [GradC]`="?" THEN 1 ELSE 0 END)/COUNT(`Lufttemperatur [GradC]`) as `Missing Lufttemperatur`,
  SUM(CASE WHEN `rel. Luftfeuchte [%]`="?" THEN 1 ELSE 0 END)/COUNT(`rel. Luftfeuchte [%]`) as `Missing rel. Luftfeuchte`, 
  SUM(CASE WHEN `Windgeschwindigkeit [m/s]`="?" THEN 1 ELSE 0 END)/COUNT(`Windgeschwindigkeit [m/s]`) as `Missing Windgeschwindigkeit`,
  SUM(CASE WHEN `Windrichtung  [Grad]`="?" THEN 1 ELSE 0 END)/COUNT(`Windrichtung  [Grad]`) as `Missing Windrichtung`,
  SUM(CASE WHEN `Windspitze [m/s]`="?" THEN 1 ELSE 0 END)/COUNT(`Windspitze [m/s]`) as `Missing Windspitze`, 
  SUM(CASE WHEN `Luftdruck [hPa]`="?" THEN 1 ELSE 0 END)/COUNT(`Luftdruck [hPa]`) as `Missing Luftdruck`,
  SUM(CASE WHEN `Sonnenscheindauer [min]`="?" THEN 1 ELSE 0 END)/COUNT(`Sonnenscheindauer [min]`) as `Missing Sonnenscheindauer`
FROM opendata_meteo_wide;
```

| Missing Lufttemperatur | Missing rel. Luftfeuchte | Missing Windgeschwindigkeit | Missing Windrichtung | Missing Windspitze | Missing Luftdruck | Missing Sonnenscheindauer |
| --- | --- | --- | --- | --- | --- | --- |
| 0.625 | 0.625 | 0.625 | 0.625 | 0.624 | 0.625 | 0.625 |

So, data quality is poor. The missing ratio is around 62% for all columns during one day of measurements (Feb 4th 2022) in 10 different locations. 

To prove that, it is possible to make a query in the former table computing the rate of missing values in the column HMW. 

```sql
SELECT
  SUM(CASE WHEN `HMW`="?" THEN 1 ELSE 0 END)/count(*) as `Portion Missing HMWs`  
FROM opendata_meteo;
```