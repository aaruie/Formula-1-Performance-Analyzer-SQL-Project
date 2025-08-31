# Formula 1 Performance Analyzer using SQL

![Formula 1 logo](https://github.com/aaruie/Formula-1-Performance-Analyzer-SQL-Project/blob/main/F1_App_Red_Logo_White_Background.avif)

## OVERVIEW

The Formula 1 Performance Analyzer is a data analytics project that uses advanced SQL to extract and analyze actionable insights from historical Formula 1 race data from the 2020 to 2024 seasons. By simulating a real-world motorsport analytics environment, this project demonstrates how data can be used to inform strategic decisions for racing teams, race engineers, and sponsors. The project focuses on applying sophisticated SQL techniques—including aggregations, window functions, and time-series analysis—to uncover performance patterns and competitive advantages that go beyond simple race results.

## OBJECTIVE

- Driver Performance Analysis: Assess driver consistency and adaptability by examining metrics like qualifying vs. race pace and points scored, especially in light of the 2022 regulation changes.
- Pit-Stop Efficiency: Measure and compare the speed of pit stops across different teams to identify the most efficient crews and analyze how pit-stop performance impacts a race's final outcome. .
- Constructor Strategy Deconstruction: Uncover team-specific strategies by correlating race results with factors like tire choice and development trends, highlighting shifts in the competitive landscape.
- Causal Race Factors: Identify key variables, such as safety cars, overtakes, and incidents, that influence a driver's final position in a race.

## DATASET

The data for this project is sourced from the Kaggle dataset:
 - Dataset : [Formula 1 Dataset](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020)

## SCHEMA

```sql

DROP TABLE IF EXISTS constructor_results;

CREATE TABLE races (
  raceId INT,
  year INT,
  round INT,
  circuitId INT,
  name varchar(30),
  date DATE
);


CREATE TABLE drivers (
  driverId INT,
  driverRef VARCHAR (30),
  number VARCHAR (4),
  code VARCHAR(4),
  forename	VARCHAR(25),
  surname VARCHAR(30),	
  dob	DATE,
  nationality VARCHAR(20)
);

CREATE TABLE results (
  resultId INT,
  raceId  INT,	
  driverId	INT,
  constructorId	INT,
  number  VARCHAR(5),
  grid	INT,
  position  VARCHAR(5),
  positionText	VARCHAR(5),
  positionOrder	INT,
  points FLOAT,
  statusId INT
);

CREATE TABLE constructors (
  constructorId  INT,
  constructorRef	VARCHAR(30),
  constructor_name	VARCHAR(30),
  nationality	VARCHAR(15)
);

CREATE TABLE lap_times (
  raceId INT,
  driverId	INT,
  lap	INT,
  position INT,
  time	VARCHAR (10),
  milliseconds INT
);

CREATE TABLE pit_stops (
  raceId  INT,
  driverId INT,
  stop	INT,
  lap	INT,
  time	VARCHAR(10),
  duration	FLOAT,
  milliseconds INT
);

CREATE TABLE constructor_results (
  constructorResultsId INT,	
  raceId  INT,
  constructorId	 INT,
  points FLOAT
);

```

## ANALYTICAL QUESTIONS AND SOLUTIONS


### 1. Who are the most consistent drivers across seasons?

**Objective :** Driver Performance & Consistency
```sql
SELECT 
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
    COUNT(r.resultId) AS races,
    ROUND(AVG(r.positionOrder), 2) AS avg_finish,
    ROUND(STDDEV_POP(r.positionOrder), 2) AS stddev_finish
FROM results_2020_2024 r
JOIN drivers d ON r.driverId = d.driverId
JOIN races ra ON r.raceId = ra.raceId
WHERE r.positionOrder IS NOT NULL
GROUP BY d.driverId, driver
ORDER BY stddev_finish ASC;
```

**Objective :**

-- 2. Which drivers gained the most positions during races?

SELECT 
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
	SUM(r.grid - r.positionOrder) AS total_positions_gained
FROM results_2020_2024 r
JOIN drivers d ON r.driverId = d.driverId
JOIN races ra ON r.raceId = ra.raceId
WHERE r.grid > 0
  AND r.positionOrder IS NOT NULL
GROUP BY d.driverId, driver
ORDER BY total_positions_gained DESC;


-- 3. Which drivers perform best at specific circuits (e.g., Monaco, Silverstone)?


SELECT 
    ra.name AS circuit_name,
    ra.year,
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
    ROUND(AVG(r.points)::numeric, 2) AS avg_points
FROM results_2020_2024 r
JOIN drivers d ON r.driverId = d.driverId
JOIN races ra ON r.raceId = ra.raceId
WHERE r.points = 25
  AND (
      ra.name ILIKE '%Monaco%' 
      OR ra.name ILIKE '%British%' 
      OR ra.name ILIKE '%Belgian%' 
      OR ra.name ILIKE '%Abu Dhabi%'
  )
GROUP BY ra.name, ra.year, d.driverId, driver
ORDER BY circuit_name, ra.year, avg_points DESC;


-- 4. Top 5 drivers with the most podium finishes (P1–P3) from 2020–2024

SELECT 
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
	d.number,
    COUNT(*) AS podiums
FROM results_2020_2024 r
JOIN drivers d ON r.driverId = d.driverId
JOIN races ra ON r.raceId = ra.raceId
WHERE r.positionOrder BETWEEN 1 AND 3
GROUP BY d.driverId, driver,d.number
ORDER BY podiums DESC
LIMIT 5;


-- 5. Driver championship ranking consistency across 5 seasons

SELECT 
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
	d.number,
    ra.year,
    COUNT(DISTINCT r.raceId) AS races_participated,
    SUM(r.points) AS total_points
FROM results_2020_2024 r
JOIN drivers d ON r.driverId = d.driverId
JOIN races ra ON r.raceId = ra.raceId
GROUP BY d.driverId, driver, d.number,ra.year
ORDER BY d.driverId, ra.year;

-- 6. Which constructors have the best average finish positions?

SELECT 
    c.constructorId,
    c.constructor_name,
    ROUND(AVG(r.positionOrder), 2) AS avg_finish_position
FROM results_2020_2024 r
JOIN constructors c ON r.constructorId = c.constructorId
JOIN races ra ON r.raceId = ra.raceId
WHERE r.positionOrder IS NOT NULL 
GROUP BY c.constructorId, c.constructor_name
ORDER BY avg_finish_position ASC;


-- 7. Which team gained the most points in the last 3 seasons?

SELECT 
    c.constructorId,
    c.constructor_name,
	ra.year,
    SUM(r.points) AS total_points
FROM results_2020_2024 r
JOIN constructors c ON r.constructorId = c.constructorId
JOIN races ra ON r.raceId = ra.raceId
GROUP BY c.constructorId, c.constructor_name,ra.year
ORDER BY total_points DESC;


-- 8. Most improved constructor year-over-year

SELECT 
    constructorId,
	constructor_name,
    year,
    SUM(points) AS yearly_points
FROM (
    SELECT 
        c.constructorId,
		c.constructor_name,
        ra.year,
        r.points
    FROM results_2020_2024 r
    JOIN constructors c ON r.constructorId = c.constructorId
    JOIN races ra ON r.raceId = ra.raceId
) AS sub
GROUP BY constructorId, constructor_name, year
ORDER BY constructorId, year;


-- 9. Team performance by track: who dominates where?
-→ Join races, results, and constructors, group by circuitId.

SELECT
   c.constructorId,
   c.constructor_name,
   ra.name AS circuit_name,
   ROUND(SUM(r.points)::numeric, 2) AS total_points
FROM results_2020_2024 r
JOIN races ra ON r.raceId = ra.raceId
JOIN constructors c ON r.constructorId = c.constructorId
GROUP BY  c.constructorId, c.constructor_name, ra.name
ORDER BY circuit_name, total_points DESC;


-- 10. Constructor championship rankings across 5 seasons

SELECT 
    c.constructor_name,
    ra.year,
    SUM(co.points) AS total_points
FROM constructor_results co
JOIN constructors c ON co.constructorId = c.constructorId
JOIN races ra ON co.raceId = ra.raceId
WHERE ra.year BETWEEN 2020 AND 2024
GROUP BY c.constructor_name, ra.year
ORDER BY ra.year, total_points DESC; 


-- 11. What’s the average pit stop duration per constructor?


SELECT 
    c.constructor_name,
    ROUND(AVG(ps.duration)::numeric, 2) AS avg_pit_duration
FROM pit_stops_2020_2024 ps
JOIN results r ON ps.raceId = r.raceId AND ps.driverId = r.driverId
JOIN constructors c ON r.constructorId = c.constructorId
WHERE ps.duration IS NOT NULL
GROUP BY c.constructor_name
ORDER BY avg_pit_duration;



-- 12. How many pit stops do drivers take per race on average?


SELECT 
    ps.driverId,
    d.forename || ' ' || d.surname AS driver,
    ROUND(AVG(stop_counta), 2) AS avg_pit_stops
FROM (
    SELECT driverId, raceId, COUNT(*) AS stop_count
    FROM pit_stops_2020_2024
    GROUP BY driverId, raceId
) ps
JOIN drivers d ON ps.driverId = d.driverId
GROUP BY ps.driverId, driver
ORDER BY avg_pit_stops DESC;


-- 13. How does pit stop count affect finishing position?

SELECT 
    r.driverId,
    d.forename,
    d.surname,
    COUNT(ps.stop) AS pit_stops,
    r.positionOrder
FROM pit_stops_2020_2024 ps
JOIN results r ON ps.raceId = r.raceId AND ps.driverId = r.driverId
JOIN drivers d ON r.driverId = d.driverId
WHERE r.positionOrder IS NOT NULL
GROUP BY r.driverId, d.forename, d.surname, r.positionOrder
ORDER BY pit_stops;


-- 14. Lap time degradation analysis (top circuits only)
-- (Analyze lap times across laps to observe performance drop over time)


SELECT 
    lt.raceId,
    lt.driverId,
	d.forename || ' ' || d.surname AS driver,
    ra.year,
    ra.circuitId,
	ra.name,
    lt.lap,
    AVG(lt.milliseconds) AS avg_lap_time
FROM lap_times_2020_2024 lt
JOIN races ra ON lt.raceId = ra.raceId
JOIN drivers d ON lt.driverId = d.driverId
WHERE ra.circuitId IN (6, 9, 15, 13, 24)
GROUP BY lt.raceId, lt.driverId,driver, ra.year,ra.name, ra.circuitId, lt.lap
ORDER BY ra.circuitId, lap;



-- 15. Who are the fastest drivers on average over entire races (only top 7 tracks)?

SELECT 
    d.driverId,
    d.forename || ' ' || d.surname AS driver,
    ROUND(AVG(lt.milliseconds), 2) AS avg_lap_time
FROM lap_times_2020_2024 lt
JOIN drivers d ON lt.driverId = d.driverId
JOIN races ra ON lt.raceId = ra.raceId
WHERE ra.circuitId IN (6, 9, 13, 15, 24)
GROUP BY d.driverId, driver
ORDER BY avg_lap_time ASC;

