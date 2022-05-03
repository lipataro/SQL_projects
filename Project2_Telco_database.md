# Project 2 - Telco database

1. There are three tables under telecommunication database, namely event, demographic and cell described as below.
    1. Event: is a table for data usage of 100 customers of a telco provider for 1 sample day of their Facebook, Instagram, YouTube, Netflix and WhatsApp.
    2. Cell: is a table of cell ids that customers were at with their latitude and longitude
    3. Demographic: is a table of subscribers with their demographics such as gender.
2. Copy these tables under your sandbox database.

```sql
CREATE TABLE cell AS SELECT * FROM telecommunication.cell; 
CREATE TABLE demographic AS SELECT * FROM telecommunication.demographic ; 
CREATE TABLE event AS SELECT * FROM telecommunication.event ;
```

1. Get familiar with the tables and number of rows and columns
2. Create a table with average duration of stay (dwell time in minutes) of each customer at each cell that customer stayed at least 5 minutes. (provide your query as well. This table should have three columns, subscriber_id, aircomcellid, duration_minutes)

```sql
CREATE TABLE cell_stay AS 
with shifted as (
      SELECT e.*,
             row_number() over ( partition by subscriber_id order by `time`)  as rn
      FROM event e  
)
SELECT subscriber_id , aircomcellid , sum(duration) as duration_minutes
FROM 
	(SELECT S1.subscriber_id , S1.aircomcellid, round((S2.time - S1.time)/60) as duration
	FROM shifted S1  
	JOIN shifted S2  
  	ON S1.rn = S2.rn - 1
  	AND S1.subscriber_id = S2.subscriber_id) res
WHERE res.duration >=5
group by res.subscriber_id,res.aircomcellid 
ORDER BY res.subscriber_id,res.aircomcellid;

SELECT * FROM cell_stay LIMIT 9;
```

| subscriber_id | aircomcellid | duration_minutes |
| --- | --- | --- |
| 02e74f10e0327ad868d138f2b4fdd6f0 | A | 225 |
| 02e74f10e0327ad868d138f2b4fdd6f0 | B | 699 |
| 02e74f10e0327ad868d138f2b4fdd6f0 | C | 426 |
| 03afdbd66e7929b125f8597834fa83a4 | A | 230 |
| 03afdbd66e7929b125f8597834fa83a4 | B | 726 |
| 03afdbd66e7929b125f8597834fa83a4 | C | 387 |
| 072b030ba126b2f4b2374f342be9ed44 | A | 190 |
| 072b030ba126b2f4b2374f342be9ed44 | B | 892 |
| 072b030ba126b2f4b2374f342be9ed44 | C | 268 |
1. Create a table with histogram which has two columns, interval and cnt_sub, in which interval is 100 minutes (0,100,200,300,400,500,600, etc.) and cnt_sub is how many subscribers stayed on average at each cell more than 5 minutes on each interval. (provide your query as well)

```sql
CREATE TABLE if not exists histogram (
`interval` bigint,
cnt_sub bigint);

INSERT INTO histogram (`interval`,cnt_sub) VALUES 
(100, (SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 0 AND 100 THEN 1 ELSE 0 END)
FROM cell_stay)),
(200,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 101 AND 200 THEN 1 ELSE 0 END)
FROM cell_stay)),
(300,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 201 AND 300 THEN 1 ELSE 0 END)
FROM cell_stay)),
(400,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 301 AND 400 THEN 1 ELSE 0 END)
FROM cell_stay)),
(500,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 401 AND 500 THEN 1 ELSE 0 END)
FROM cell_stay)),
(600,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 501 AND 600 THEN 1 ELSE 0 END)
FROM cell_stay)),
(700,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 601 AND 700 THEN 1 ELSE 0 END)
FROM cell_stay)),
(800,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 701 AND 800 THEN 1 ELSE 0 END)
FROM cell_stay)),
(900,(SELECT
  SUM(CASE WHEN duration_minutes BETWEEN 801 AND 900 THEN 1 ELSE 0 END)
FROM cell_stay))
;

SELECT * FROM histogram;
```

| interval | cnt_sub |
| --- | --- |
| 100 | 1 |
| 200 | 21 |
| 300 | 55 |
| 400 | 67 |
| 500 | 38 |
| 600 | 36 |
| 700 | 46 |
| 800 | 27 |
| 900 | 9 |
1.  Which customer_id has longest avg dwell time (don’t consider stays at cells that were less than 5 minutes) and which customer_id has shortest.

```sql
# Shortest average dwell time 
SELECT * , 
row_number() over (order by duration_minutes) as rnk 
FROM cell_stay limit 1;
```

| subscriber_id | aircomcellid | duration_minutes | rnk |
| --- | --- | --- | --- |
| b6d767d2f8ed5d21a44b0e5886680cb9 | A | 94 | 1 |

```sql
# Longest average dwell time
SELECT * , 
row_number() over (order by -duration_minutes) as rnk 
FROM cell_stay limit 1;
```

| subscriber_id | aircomcellid | duration_minutes | rnk |
| --- | --- | --- | --- |
| 35f4a8d465e6e1edc05f3d8ab658c551 | B | 893 | 1 |
1. What is the total bytesdown and bytesup usage of netflix between 8 to 10 pm? Compare this usage with netflix usage between 8 and 10 am.

```sql
SELECT SUM(bytesdown), SUM(bytesup) FROM event
WHERE service ='Netflix' AND `hour` BETWEEN '20' and '21'; #Higher in the evening
```

| SUM(bytesdown) | SUM(bytesup) |
| --- | --- |
| 43896969 | 780130 |

```sql
SELECT SUM(bytesdown), SUM(bytesup) FROM event
WHERE service ='Netflix' AND `hour` BETWEEN '8' and '9'; #Lower in the morning
```

| SUM(bytesdown) | SUM(bytesup) |
| --- | --- |
| 7529198 | 381962 |

If you want to have the solution in just one query (table), then:

```sql
SELECT
	(CASE
    	WHEN `hour` BETWEEN '8' and '9' THEN 'morning'
			WHEN `hour` BETWEEN '20' and '21' THEN 'evening'
    END) moment_of_day,
    SUM(bytesdown) AS total_bytes_down,
    SUM(bytesup) AS total_bytes_up
FROM event
WHERE service ='Netflix'
GROUP BY moment_of_day
HAVING moment_of_day IS NOT NULL;
```

| moment_of_day | total_bytes_down | total_bytes_up |
| --- | --- | --- |
| evening | 43896969 | 780130 |
| morning | 7529198 | 381962 |
1. On average which service has highest bytesdown?

```sql
SELECT service, round(avg(sb.bytesdown)) AS avg_bytesdown FROM (
	SELECT service, bytesdown ,
	RANK () over (partition by service order by -bytesdown) as rnk 
	FROM event) sb
GROUP BY service
ORDER BY avg_bytesdown DESC;
```

| service | avg_bytesdown |
| --- | --- |
| Instagram | 383860 |
| Netflix | 156964 |
| Facebook | 101479 |
| YouTube | 80166 |
| WhatsApp | 7748 |
1. Which cell_id has highest load (bytesdown + bytesup) on Youtube?

```sql
SELECT SUM(CASE WHEN service = 'YouTube' THEN bytesup + bytesdown  ELSE 0 END) AS Youtube_load,
	aircomcellid 
FROM event
GROUP BY aircomcellid 
ORDER BY Youtube_load DESC LIMIT 1;  
```

| Youtube_load | aircomcellid |
| --- | --- |
| 82237170 | B |
1. Which hour of the day we have highest data consumption on each cells? Do all cells are on highest peak at the same hour of the day?

```sql
SELECT `hour`, aircomcellid, total_load AS max_load
	FROM (SELECT *,
	RANK() OVER (PARTITION BY aircomcellid ORDER BY -total_load) AS rnk
	FROM
		(SELECT
		`hour`,
		aircomcellid,
		sum(bytesdown+bytesup) as total_load
		FROM event
		GROUP BY `hour`, aircomcellid
		ORDER BY aircomcellid) cell_load) ranked
WHERE ranked.rnk = 1;
# The cells have different peak hours, although always in the evening.
```

| hour | aircomcellid | max_load |
| --- | --- | --- |
| 21 | A | 24798354 |
| 20 | B | 57332524 |
| 23 | C | 33127868 |
1. What portion of males and females use Netflix? 

```sql
SELECT d.gender, COUNT(DISTINCT(e.subscriber_id)) AS `USE_NETFLIX [%]`  
FROM event e
JOIN demographic d  ON e.subscriber_id = d.subscriber_id 
WHERE service = 'Netflix'
GROUP BY gender ;   #Since count(subscribers)=100, no need for calculating %
```

| gender | USE_NETFLIX [%] |
| --- | --- |
| Female | 47 |
| Male | 53 |

What portion of males and females use instagram? 

```sql
SELECT d.gender, COUNT(DISTINCT(e.subscriber_id)) AS `USE_INSTAGRAM [%]`  
FROM event e
JOIN demographic d  ON e.subscriber_id = d.subscriber_id 
WHERE service = 'Instagram'
GROUP BY gender ;   #Since count(subscribers)=100, no need for calculating %
```

| gender | USE_INSTAGRAM [%] |
| --- | --- |
| Female | 47 |
| Male | 53 |

Which service is mostly used (duration) by males?

```sql
with shifted as (
      SELECT e.*,
             row_number() over ( partition by subscriber_id order by `time`) as rn
      FROM event e  
)
SELECT res.service, sum(res.duration) as duration_minutes, d.gender
FROM 
	(SELECT S1.subscriber_id , S1.service, round((S2.time - S1.time)/60) as duration
	FROM shifted S1  
	JOIN shifted S2  
  	ON S1.rn = S2.rn - 1
  	AND S1.subscriber_id = S2.subscriber_id) res
  	JOIN demographic d ON res.subscriber_id = d.subscriber_id 
WHERE res.duration >=5 AND d.gender = 'MALE'
group by res.service 
ORDER BY duration_minutes desc;
# Instagram is the most used by Males.
```

| service | duration_minutes | gender |
| --- | --- | --- |
| Instagram | 20901 | Male |
| YouTube | 15128 | Male |
| WhatsApp | 14462 | Male |
| Netflix | 11138 | Male |
| Facebook | 10356 | Male |