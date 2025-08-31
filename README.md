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

-- Create and load the full 'races', 'drivers', 'results', 'constructors', 'lap_times', 'pit_stops' table--

'''sql

DROP TABLE IF EXISTS constructor_results;
CREATE TABLE races (
  raceId INT,
  year INT,
  round INT,
  circuitId INT,
  name varchar(30),
  date DATE
);
'''

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


