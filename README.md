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
- [Next Steps](#next-steps)

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

###### Data Model
![Data Model](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Data%20model.png)

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

## Insights

Water Access Challenges and Metrics
###### Snapshot of Distribution
![Distribution](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Water%20source%20distribution.png)

## Insights on Water Distribution
1. Urban vs. Rural Distribution
Problem: Urban areas serve 10M people (36.15%), while rural areas, with limited infrastructure, serve 18M (63.85%) of the population, creating a significant disparity.
Solution: Invest in rural water infrastructure, such as wells and communal taps, to balance access.
Key Metric: Rural areas, serving nearly two-thirds of the population (63.85%), face greater challenges in water source reliability.

2. Water Source Utilization
Problem: Wells, the most utilized source, serves 10.9K people (46%). Shared taps serve 4.3K (18.1%), while broken home taps impact 5M (21%), indicating inefficient infrastructure.
Solution: Focus on repairing broken taps and improving access to reduce dependency on shared taps.
Key Metric: Shared taps and "Tap in Home Broken" together serve 38.1% of the population, representing critical areas for intervention.

3. City-Level Distribution
Problem: Larger cities like Harare (10.5%) have better water source distribution than smaller cities like Ilanga, exacerbating disparities.
Solution: Target smaller cities such as Ilanga and Kintampo with tailored water access programs.
Key Metric: Harare serves 10.5% of "Tap in Home Broken" users, while smaller cities account for less than 5% each.

4. Tap in Home Broken
Problem: Broken taps serve 5M people (21%), leading to inefficiency, delays, and health risks.
Solution: Introduce regular maintenance schedules and fast repair response systems for taps in urban and rural areas.
Key Metric: Harare (10.5%) and Amina (16%) account for the largest shares of broken tap users, underscoring systemic maintenance issues.

5. Geographic Patterns
Problem: Water access is uneven, with regions like Harare being well-connected while outlying areas are underserved. Some regions serve fewer than 1% of the population per water source.
Solution: Focus on underserved regions to ensure equitable water access across all locations.
Key Metric: Harare serves 10.5% of "Tap in Home Broken" users, while outlying regions serve significantly lower percentages.

6. Community Impact
Problem: Reliance on broken or shared water sources affects over 21% of the population, increasing inefficiencies, waiting times, and health risks.
Solution: Develop sustainable water systems using technology for real-time issue reporting and improved management.
Key Metric: Over 21% of the population depends on broken taps, and 10% on rivers, causing significant delays and inefficiencies in water collection.


###### Snapshot of Gender disparities
![Gender Disparity](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Gender%20disparity.png)

## Insights on Gender Disparity and Crime Data
1. Gender Disparity in Crime Victimization
Problem: Males are the most affected group, making up 64.39% of all crime victims (49.72K victims). Women, at 25.42% (19.63K victims), are disproportionately targeted in harassment and sexual assault cases, while children face vulnerabilities across various crimes.
Solution: Implement targeted safety measures for men during peak crime hours and develop specialized support systems for women and children, particularly in high-risk areas like Kilimani.
Key Metric: Males constitute 64.39% of crime victims, while 80% of sexual assault victims are female.

2. Gender Disparities in Crime Types
Problem: Gender-based crime patterns show significant disparities. Women are predominantly victims of harassment (60%) and sexual assault (80%), while men are most affected by theft (70%) and public intoxication crimes (65%).
Solution: Launch gender-specific interventions, such as community awareness programs, female-focused safety networks, and male-targeted theft prevention initiatives.
Key Metric: Women account for 60% of harassment victims, and men are the majority (70%) in theft cases.

3. Provincial Crime Distribution
Problem: Urban areas like Kilimani report the highest crime rates (20K crimes), with males consistently representing the majority of victims (~65%). Outlying provinces like Hawassa experience lower crime rates but maintain gender disparities.
Solution: Focus resources and law enforcement in high-crime areas like Kilimani while addressing rural crime prevention with tailored strategies.
Key Metric: Kilimani accounts for 20K crimes, with males constituting ~65% of victims.

4. Crime Patterns by Hour
Problem: Crime peaks between 8 PM and 10 PM, predominantly affecting males (~70% of victims during these hours), indicating risks associated with mobility and activities during nighttime.
Solution: Enhance nighttime policing, increase lighting in urban areas, and encourage community patrol initiatives during peak hours.
Key Metric: 70% of nighttime crime victims are male, with incidents spiking from 8 PM to 10 PM.

5. Daily Crime Trends
Problem: Mondays and Fridays experience the highest crime rates, affecting all genders but disproportionately males (~65%). This may be linked to work-related activities or heightened mobility on these days.
Solution: Implement strategic scheduling of law enforcement patrols and public awareness campaigns to mitigate risks on high-crime days.
Key Metric: Mondays and Fridays see the highest crime occurrences, with ~65% of victims being male.

6. Gender-Disparity-Related Crimes in Water Collection
Problem: Women face 70% of harassment incidents during water collection, making them highly vulnerable in areas where water access is limited or requires long travel.
Solution: Introduce gender-sensitive interventions such as security patrols near water collection points and improved infrastructure to reduce travel distance.
Key Metric: 70% of harassment victims during water collection are women.

7. Community Safety and Vulnerable Groups
Problem: Children, while constituting only 10.2% of victims, remain a vulnerable group due to their inability to defend themselves or navigate risks effectively.
Solution: Develop child protection programs and integrate safety education into schools and communities to mitigate risks.
Key Metric: Children make up 10.2% of crime victims, with incidents occurring sporadically across all time periods.

###### Snapshot of Queue composition
![Queue composition](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Queue%20composition%20days.png)

## Insights on Queue Composition for Shared Taps
1. Queue Demographics
Problem: Women dominate queue composition, making up 66%, followed by men at 24% and children at 10%. This indicates gender disparities in water collection responsibilities.
Solution: Develop gender-balanced solutions, such as equitable household water collection schedules or targeted support for women.
Key Metric: Women represent 66% of those queuing for shared taps.

2. Regional Queue Burdens
Problem: Kilimani has the highest time spent in queues, totalling 702 days, significantly higher than Hawassa (340 days). This reflects regional disparities in shared tap availability and efficiency.
Solution: Increase the number of shared taps or improve their distribution in high-demand areas like Kilimani.
Key Metric: Kilimani accounts for 702 days spent in queues, compared to 340 days in Hawassa.

3. Daily Queue Times
Problem: Saturdays experience the longest average queue times at 246 minutes, likely due to increased demand from weekend activities, compared to Sundays at 82 minutes.
Solution: Extend operational hours or allocate additional water sources on peak days to reduce waiting times.
Key Metric: Queue times on Saturdays are 3x longer than on Sundays.

4. Hourly Queue Trends
Problem: Queue times peak between 4 PM and 6 PM, suggesting that late afternoon is the busiest period for water collection.
Solution: Optimize water distribution schedules to meet higher demand during peak hours.
Key Metric: Peak hourly queue times exceed 250 minutes, significantly above off-peak times.

5. Impact on Productivity
Problem: Long queues, especially on peak days and times, lead to a loss of productive hours, disproportionately affecting women who spend the most time queuing.
Solution: Introduce time-saving interventions such as community queue management systems or priority access for certain groups.
Key Metric: Women, who make up 66% of the queue, are most affected by 246-minute Saturday queues.

6. Children’s Vulnerability
Problem: Though children make up only 10% of the queues, long wait times expose them to risks, such as reduced study time and health hazards.
Solution: Provide safe waiting areas or supervised collection systems for children.
Key Metric: Children spend significant time in queues despite being a minority (10%).


###### Snapshot of Progress after 1-year
![One Year Dashboard](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/After%201%20year.png)

## Insights on Project Progress
1. Population with Basic Water Access
At Onset: Only 33.59% of the population had basic access, benefiting 11K people.
After One Year: Access increased to 48%, positively impacting 3.94M people.
Insight: A 14.41% increase in access highlights significant progress, reaching over 3.93M additional people in the first year.

2. Project Completion
At Onset: 0% project completion with 25,369 water sources to address.
After One Year: 22% of the project was completed, reducing the remaining sources to 19,910.
Insight: Approximately 5,459 sources were improved within the first year, marking a strong start.

3. Budget and Costs
At Onset: Total costs were $131,915, slightly exceeding the $128,450 budget (-2.7%).
After One Year: Costs surged to $33.79M, surpassing the $30.52M budget by 10.69%.
Insight: While the increase reflects project expansion, budget overruns indicate cost management challenges.

4. Cost Distribution by Province
At Onset: Provincial cost distribution was relatively balanced.
After One Year: Kilimani ($11.1M, 32.84%) and Sokoto ($6.66M, 19.7%) became cost-intensive focus areas.
Insight: High expenditures in key provinces demonstrate concentrated efforts where needs are greatest.

5. Aggregated Improvements
At Onset: The focus was primarily on reverse osmosis (RO) filter installations ($130K).
After One Year: Expanded activities included $14.3M for RO filters, $9.4M for public taps, $7.7M for well drilling, $1.7M for UV/RO installations, and $0.7M for repairs.
Insight: Diversification of improvement efforts addressed broader water access challenges.

6. Key Achievements
People Helped: The number of beneficiaries increased from 11K to 3.94M, showing a massive scale-up in impact.
Project Reach: Completion rates improved, and costs were allocated effectively to expand reach.

Overall Recommendations
- Focus on Underserved Regions: Direct efforts to provinces with minimal progress to ensure equitable access to water resources.
- Enhance Budget Management: Adopt cost-saving strategies and periodic financial audits to control overruns.
- Accelerate Implementation: Introduce streamlined workflows, increase field staff, and use technology to expedite project execution.
- Engage Communities: Foster local involvement to improve infrastructure maintenance, reporting, and long-term sustainability.
- Monitor Progress Regularly: Use data-driven approaches to measure impact, identify challenges, and recalibrate strategies as needed.


## Next Steps
The Maji Ndogo project exemplifies how data-driven approaches can effectively tackle real-world challenges. By integrating SQL and Power BI, I demonstrated actionable insights to improve water access, enhance budget efficiency, and address societal disparities.
