# Visualizing Maji Ndogo's Water Transformation

## Table of Contents
- [Overview](#overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [SQL Workflow](#sql-workflow)
- [Key SQL Scripts](#key-sql-scripts)
- [Power BI Workflow](#power-bi-workflow)
- [Key Power BI Scripts](#key-power-bi-scripts)
- [Insights](#insights)
- [Conclusion](#conclusion)

## Overview
The project leverages data analytics to uncover insights about water availability, infrastructure needs, population dynamics, and socio-economic impacts in addressing critical water access challenges. By combining SQL for data cleaning and transformation with Power BI for advanced visualization, I provide actionable intelligence to guide decision-makers and stakeholders.

**Goals**
- Access and Equity: Improve water access and address gender disparities.
- Sustainability: Optimize budget usage and reduce water-source-related crime.
- Stakeholder Empowerment: Provide actionable insights through interactive dashboards.

###### Snapshot of Project at Onset
![Project Onset](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Onset.png)

## Data Source
-	ALX: Survey data on water sources and demographics.
-	ALX: Budget and cost details for infrastructure improvements.


## Tools
-	SQL: Data extraction, cleaning, and analysis.
-	Power BI: Data modelling, dashboard design, and reporting.


## SQL Workflow

Data Cleaning
  -	Removed duplicates.
  -	Null and Blank values.
  -	Standardized column data types for consistency.
  -	Performed UPDATES

Transformation and Analysis
  -	Implemented JOINS, CTEs, and Window Functions for aggregations and insights.
  -	Used subqueries to identify high-priority regions for interventions.
  -	Created Temporary Tables and Views for intermediary calculations and visualization.


## Key SQL Scripts
**1.	Explore Database:**
```ruby
    SHOW TABLES;
```

```ruby
    SELECT *
    FROM location LIMIT 5;  
```

**2.	Clean Employee Data:**
```ruby
    UPDATE employee  
    SET email = CONCAT(LOWER(REPLACE(employee_name, ' ', '.')), '@ndogowater.gov');  
```
```ruby
    UPDATE
       well_pollution
    SET
       description = 'Bacteria: Giardia Lamblia'
    WHERE
       description = 'Clean Bacteria: Giardia Lamblia';
```
```ruby
    UPDATE
       well_pollution
    SET
       results = 'Contaminated: Biological'
    WHERE
       biological > 0.01 AND results = 'Clean';
```

**3.	Join for Analysis:**
```ruby
    SELECT  
      location.province_name,  
      location.town_name,  
      water_source.type_of_water_source,  
      visits.time_in_queue  
    FROM visits  
      INNER JOIN location ON visits.location_id = location.location_id  
      INNER JOIN water_source ON visits.source_id = water_source.source_id;  
```

**4.	Combined View:**
```ruby
    CREATE VIEW combined_analysis_table AS  
      SELECT  
        water_source.type_of_water_source,  
        location.town_name,  
        location.province_name,  
        location.location_type,  
        water_source.number_of_people_served,  
        visits.time_in_queue,  
        well_pollution.results  
      FROM visits  
        LEFT JOIN well_pollution ON well_pollution.source_id = visits.source_id  
        INNER JOIN location ON location.location_id = visits.location_id  
        INNER JOIN water_source ON water_source.source_id = visits.source_id;
```

**5. CTEs:**
```ruby
    WITH Incorrect_records AS(
    SELECT
      a.location_id AS location_id,
      v.record_id,
      e.employee_name,
      a.true_water_source_score AS auditor_score,
      wq.subjective_quality_score AS surveyor_score
    FROM 
      auditor_report a
    JOIN 
      visits v
      ON a.location_id = v.location_id
    JOIN
      water_quality wq
      ON v.record_id = wq.record_id
    JOIN 
      employee e
      ON v.assigned_employee_id = e.assigned_employee_id
    WHERE v.visit_count = 1 AND
      a.true_water_source_score != wq.subjective_quality_score
      )
    SELECT *
    FROM Incorrect_records;
```

**6. Window Functions:**
```ruby
    SELECT 
      location_id,
      ROW_NUMBER() OVER (PARTITION BY province ORDER BY population_served DESC) AS Rank
    FROM visits;
```
```ruby
    SELECT 
      source_id,
      type_of_water_source, 
      SUM(number_of_people_served) AS total_people_served,
      RANK() OVER (ORDER BY SUM(number_of_people_served) DESC) AS ranka
    FROM
      water_source
    WHERE type_of_water_source <> "tap_in_home"
    GROUP BY source_id
    ORDER BY total_people_served DESC;
```

**7. Case Statements:**
```ruby
    SELECT 
      TIME_FORMAT(TIME(time_of_record), '%H:00') AS hour_of_day,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Sunday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Sunday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Monday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Monday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Tuesday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Tuesday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Wednesday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Wednesday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Thursday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Thursday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Friday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Friday,
      ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Saturday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Saturday
    FROM
      visits
    WHERE
      time_in_queue != 0
    GROUP BY hour_of_day
    ORDER BY hour_of_day;
```

## Power BI Workflow
-	Imported clean datasets.
-	Data modelling.
-	Visualized key metrics with DAX Measures like cumulative_budget and population_with_basic_access.
-	Drill-down capabilities from national to town levels.

## Key Power BI Scripts
**1. DAX Measures:**
```ruby
     Rural_Adjusted_Cost = infrastructure_cost[unit_cost_USD] * 1.5
```
```ruby
     Basic_water_access_level = 
      DIVIDE(
            CALCULATE(SUM(water_source[number_of_people_served]), water_source[Basic_water_access] == "Basic Access"),
            SUM(water_source[number_of_people_served])
            )
```

**2. DAX Columns:**
```ruby
     Average_queue_time = 
             CALCULATE(
             AVERAGE('visits'[time_in_queue]),                 
             FILTER(
             'visits',
             'visits'[source_id] = 'water_source'[source_id])    
              )
```
```ruby
     Basic_water_access = 
              IF(
               AND(                                             
               water_source[type_of_water_source] == "well",
               RELATED(well_pollution[results]) == "Clean"),
               "Basic Access",                                  
               IF(                                              
               water_source[type_of_water_source] == "tap_in_home",
               "Basic Access",                                  
               IF(
               AND(                                             
               water_source[type_of_water_source] == "shared_tap",
               water_source[Average_queue_time] < 30),
               "Basic Access",                                  
               "Below Basic Access"                             
               )
               )
               )
```
###### Snapshot of Progress after 1-year
![One Year Dashboard](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/After%201%20year.png)

## Insights
### National Level
•	Problem: Uneven distribution of water sources with rural areas lagging in access.

•	Solution: Budget realignment for rural-focused projects.

•	Key Metric: 34% of the population has access to basic water sources, which is projected to rise to 57% with planned improvements.

###### Snapshot of Distribution
![Distribution](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Water%20source%20distribution.png)

### Provincial and Regional Level
•	Problem: Gender disparities in queue times and water access.

•	Solution: Targeted interventions for areas with significant disparities.

•	Key Metric: Over 60% of women in rural areas spend more time collecting water than their urban counterparts.

###### Snapshot of Gender disparities
![Gender Disparity](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Gender%20disparity.png)

### Town Level
•	Problem: Rising crime rates near water sources in certain towns.

•	Solution: Community policing initiatives tied to water infrastructure improvements.

•	Key Metric: Reduction in water-source-related crimes after infrastructure upgrades.

###### Snapshot of Queue composition
![Queue composition](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Queue%20composition%20days.png)


## Conclusion
The Maji Ndogo project exemplifies how data-driven approaches can effectively tackle real-world challenges. By integrating SQL and Power BI, I demonstrated actionable insights to improve water access, enhance budget efficiency, and address societal disparities.
