# Visualizing Maji Ndogo's Water Transformation

## Table of Contents
- [Overview](#overview)
- [Executive Summary](#executive-summary)
- [Key Questions](#key-questions)
- [Data Source](#data-source)
- [Tools](#tools)
- [SQL Workflow](#sql-workflow)
- [Key SQL Scripts](#key-sql-scripts)
- [Power BI Workflow](#power-bi-workflow)
- [Key Power BI Scripts](#key-power-bi-scripts)
- [Insights](#insights)
- [Next Steps](#next-steps)

## Overview

Clean water is not just a resource; it is the foundation of life, health, and opportunity. Yet, for millions of people worldwide, access to clean water remains a distant dream. In underserved communities, the daily struggle for water defines life — women and children trek for miles, livelihoods suffer, and progress is stifled by inequities.

Maji Ndogo's Water Transformation Project envisions a world where clean water is not a privilege but a right—where every community, regardless of location, has equitable and sustainable access to safe water. This vision drives its mission to identify, address, and resolve the unique challenges faced by water-scarce communities through the power of data and innovation.

Using Power BI and SQL, the project uncovers stories hidden in the numbers—mapping regional disparities, analyzing costs, and crafting cost-effective infrastructure solutions. These insights enable stakeholders to prioritize underserved areas, allocate resources effectively, and implement tailored solutions for maximum impact.

**Goals**

The primary goal of the project is to ensure equitable and sustainable access to clean water by:
- Identifying cost-effective water infrastructure solutions.
- Reducing disparities in water access across different regions.
- Enhancing decision-making through data visualization and insights.
- Monitoring and optimizing the implementation of water-related projects.

###### Snapshot of Project at Onset
![Project Onset](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Onset.png)

## Executive Summary

The Maji Ndogo water transformation initiative has made significant strides in addressing water access challenges, with key insights highlighting the progress and areas requiring further attention. At the onset, only 33.59% of the population—approximately 11,000 people—had basic water access. Within a year, this figure rose to 48%, benefiting an additional 3.93 million individuals. The project achieved 22% completion, improving 5,459 water sources out of the initial 25,369, underscoring its strong start despite facing regional disparities and infrastructure inefficiencies.

Rural areas remain disproportionately underserved, with 63.85% of the population depending on limited infrastructure compared to 36.15% in urban regions. Wells, serving 46% of the population, are the most utilized water source, but broken home taps impact 5 million people (21%), further exacerbating inefficiencies. Cities like Harare show better water source distribution than smaller urban centres such as Ilanga, highlighting the need for targeted investments in underserved locations. Addressing these disparities requires repairing infrastructure, expanding access to shared taps, and improving maintenance systems.

The project also tackled gender and time disparities. Women, who make up 66% of those in water collection queues, face significant productivity losses and heightened risks, particularly during peak hours (4–6 PM) and days like Saturdays, with average queue times of 246 minutes. Gender-sensitive interventions, such as equitable household schedules and security measures, are crucial to alleviate these challenges. Furthermore, children, who constitute 10% of queue demographics, remain vulnerable, emphasizing the importance of safe waiting areas and supervised collection systems.

Despite notable achievements, the project experienced cost management challenges. Initial expenses of $131,915 slightly exceeded the $128,450 budget, but costs surged to $33.79 million within a year, surpassing the $30.52 million budget by 10.69%. Provinces like Kilimani ($11.1 million) and Sokoto ($6.66 million) accounted for the largest expenditures, reflecting focused efforts in high-need areas. Diversified improvements, such as $14.3 million for reverse osmosis filters and $9.4 million for public taps have significantly enhanced water access, but strategic cost control remains a priority to sustain progress.


## Key Questions

- Where are the longest queue times at water sources, and what does that reveal about demand across Maji Ndogo?

- Which regions have the most contaminated water sources, and how severe is the contamination?

- Are certain types of water sources more prone to pollution or long queues than others?

- How evenly are water sources distributed across provinces or communities?

- Which water sources are underutilized, and where might there be opportunities for better allocation?

- How does water source usage correlate with contamination or pollution levels?

- What water quality trends can help prioritize resource investment or health interventions?

- Which locations should be prioritized for new infrastructure or rehabilitation efforts?

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
###### Snapshot of Water Distribution
![Distribution](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Water%20source%20distribution.png)

### Insights
1. **Urban vs. Rural Distribution**
   - Problem: Urban areas serve 10M people (36.15%), while rural areas, with limited infrastructure, serve 18M (63.85%) of the population, creating a significant disparity.
   - Solution: Invest in rural water infrastructure, such as wells and communal taps, to balance access.

2. **Water Source Utilization**
   - Problem: Wells, the most utilized source, require repairs for ~11K units (46%). Shared taps account for 4.3K units (18.1%), while broken home taps impact 5M (21%), indicating inefficient infrastructure.
   - Solution: Focus on repairing broken home taps to restore access for millions while improving well and shared tap infrastructure to reduce dependency on communal sources and enhance overall water accessibility.

3. **City-Level Distribution**
   - Problem: Larger cities like Harare (10.5%) have better water source distribution than smaller cities like Ilanga, exacerbating disparities.
   - Solution: Target smaller cities such as Ilanga and Kintampo with tailored water access programs.

4. **Tap in Home Broken**
   - Problem: Broken taps serve 5M people (21%), leading to inefficiency, delays, and health risks.
   - Solution: Introduce regular maintenance schedules and fast repair response systems for taps in urban and rural areas.

5. **Geographic Patterns**
   - Problem: Water access is uneven, with regions like Harare being well-connected while outlying areas are underserved. Some regions serve fewer than 1% of the population per water source.
   - Solution: Focus on underserved regions to ensure equitable water access across all locations.

6. **Community Impact**
   - Problem: Reliance on broken or shared water sources affects over 21% of the population, increasing inefficiencies, waiting times, and health risks.
   - Solution: Develop sustainable water systems using technology for real-time issue reporting and improved management.

###### Snapshot of Gender disparity and Crime
![Gender Disparity](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Gender%20disparity.png)

### Insights
1. **Gender Disparity in Crime Victimization**
   - Problem: Males are the most affected group, making up 64.39% of all crime victims (49.72K victims). Women, at 25.42% (19.63K victims), are disproportionately targeted in harassment and sexual assault cases, while children face vulnerabilities across various crimes.
   - Solution: Implement targeted safety measures for men during peak crime hours and develop specialized support systems for women and children, particularly in high-risk areas like Kilimani.

2. **Gender Disparities in Crime Types**
   - Problem: Gender-based crime patterns show significant disparities. Women are predominantly victims of harassment (60%) and sexual assault (80%), while men are most affected by theft (70%) and public intoxication crimes (65%).
   - Solution: Launch gender-specific interventions, such as community awareness programs, female-focused safety networks, and male-targeted theft prevention initiatives.

3. **Provincial Crime Distribution**
   - Problem: Urban areas like Kilimani report the highest crime rates (20K crimes), with males consistently representing the majority of victims (~65%). Outlying provinces like Hawassa experience lower crime rates but maintain gender disparities.
   - Solution: Focus resources and law enforcement in high-crime areas like Kilimani while addressing rural crime prevention with tailored strategies.

4. **Crime Patterns by Hour**
   - Problem: Crime peaks between 8 PM and 10 PM, predominantly affecting males (~70% of victims during these hours), indicating risks associated with mobility and activities during nighttime.
   - Solution: Enhance nighttime policing, increase lighting in urban areas, and encourage community patrol initiatives during peak hours.

5. **Daily Crime Trends**
   - Problem: Mondays and Fridays experience the highest crime rates, affecting all genders but disproportionately males (~65%). This may be linked to work-related activities or heightened mobility on these days.
   - Solution: Implement strategic scheduling of law enforcement patrols and public awareness campaigns to mitigate risks on high-crime days.

6. **Gender-Disparity-Related Crimes in Water Collection**
   - Problem: Women face 70% of harassment incidents during water collection, making them highly vulnerable in areas where water access is limited or requires long travel.
   - Solution: Introduce gender-sensitive interventions such as security patrols near water collection points and improved infrastructure to reduce travel distance.

7. **Community Safety and Vulnerable Groups**
   - Problem: Children, while constituting only 10.2% of victims, remain a vulnerable group due to their inability to defend themselves or navigate risks effectively.
   - Solution: Develop child protection programs and integrate safety education into schools and communities to mitigate risks.

###### Snapshot of Queue Composition for Shared Taps
![Queue composition](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Queue%20composition%20days.png)

### Insights
1. **Queue Demographics**
   - Problem: Women dominate queue composition, making up 66%, followed by men at 24% and children at 10%. This indicates gender disparities in water collection responsibilities.
   - Solution: Develop gender-balanced solutions, such as equitable household water collection schedules or targeted support for women.

2. **Regional Queue Burdens**
   - Problem: Kilimani has the highest time spent in queues, totalling 702 days, significantly higher than Hawassa (340 days). This reflects regional disparities in shared tap availability and efficiency.
   - Solution: Increase the number of shared taps or improve their distribution in high-demand areas like Kilimani.

3. **Daily Queue Times**
   - Problem: Saturdays experience the longest average queue times at 246 minutes, likely due to increased demand from weekend activities, compared to Sundays at 82 minutes.
   - Solution: Extend operational hours or allocate additional water sources on peak days to reduce waiting times.

4. **Hourly Queue Trends**
   - Problem: Queue times peak between 4 PM and 6 PM, suggesting that late afternoon is the busiest period for water collection.
   - Solution: Optimize water distribution schedules to meet higher demand during peak hours.

5. **Impact on Productivity**
   - Problem: Long queues, especially on peak days and times, lead to a loss of productive hours, disproportionately affecting women who spend the most time queuing.
   - Solution: Introduce time-saving interventions such as community queue management systems or priority access for certain groups.

6. **Children’s Vulnerability**
   - Problem: Though children make up only 10% of the queues, long wait times expose them to risks, such as reduced study time and health hazards.
   - Solution: Provide safe waiting areas or supervised collection systems for children.


###### Snapshot of Project Progress after 1-year
![One Year Dashboard](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/After%201%20year.png)

### Insights
1. Population with Basic Water Access
   - At Onset: Only 33.59% of the population had basic access, benefiting 11K people.
   - After One Year: Access increased to 48%, positively impacting 3.94M people.
   - Insight: A 14.41% increase in access highlights significant progress, reaching over 3.93M additional people in the first year.

2. Project Completion
   - At Onset: 0% project completion with 25,369 water sources to address.
   - After One Year: 22% of the project was completed, reducing the remaining sources to 19,910.
   - Insight: Approximately 5,459 sources were improved within the first year, marking a strong start.

3. Budget and Costs
   - At Onset: Total costs were $131,915, slightly exceeding the $128,450 budget (-2.7%).
   - After One Year: Costs surged to $33.79M, surpassing the $30.52M budget by 10.69%.
   - Insight: While the increase reflects project expansion, budget overruns indicate cost management challenges.

4. Cost Distribution by Province
   - At Onset: Provincial cost distribution was relatively balanced.
   - After One Year: Kilimani ($11.1M, 32.84%) and Sokoto ($6.66M, 19.7%) became cost-intensive focus areas.
   - Insight: High expenditures in key provinces demonstrate concentrated efforts where needs are greatest.

5. Aggregated Improvements
   - At Onset: The focus was primarily on reverse osmosis (RO) filter installations ($130K).
   - After One Year: Expanded activities included $14.3M for RO filters, $9.4M for public taps, $7.7M for well drilling, $1.7M for UV/RO installations, and $0.7M for repairs.
   - Insight: Diversification of improvement efforts addressed broader water access challenges.

6. Key Achievements
   - People Helped: The number of beneficiaries increased from 11K to 3.94M, showing a massive scale-up in impact.
   - Project Reach: Completion rates improved, and costs were allocated effectively to expand reach.

###### Snapshot of Cost Overrun Drivers
![Cost Overrun Drivers](https://github.com/OOMoses/Visualizing-Maji-Ndogo-s-Water-Transformation/blob/main/assets/images/Cost%20drivers.png)

### Insights
1. Improvement Type: Install Taps Nearby

   This is the most significant driver of cost increases.

   Installing 8 taps nearby raises the average cost by $14,340 above the baseline of $5,686.16, 7 taps increase the average cost by $12,060, and 4 taps result in an average cost increase of $4,070. The high cost could be due to the scale of installation, materials, or labour involved in setting up multiple taps.

2. Improvement Type: Drill Well

   Drilling a well increases the average cost by $5,840, making it significantly cheaper than installing multiple taps but still a notable contributor to overall costs. This could involve expenses related to equipment, geological surveys, and labour.

3. Location: Rural_Kilimani and Rural_Sokoto

   Costs increased by $3,710 in Rural_Kilimani and $3,420 in Rural_Sokoto compared to the baseline. These increases may be due to factors such as logistical challenges, material transportation, or local economic conditions.

4. Time Difference (e.g., Project Timeline)

   A time difference of 8.95 minutes increases the average cost by $3,620. This suggests that delays or extended timelines may add significant costs, likely due to increased labour, equipment rental, or administrative expenses.

5. Location: Rural_Akatsi

   Costs increase by $2,390 in Rural_Akatsi, making it the least impactful of the top influencers.

## Next Steps
Building on the insights gathered during this project, we are focusing on actionable improvements to streamline operations, reduce costs, and enhance overall efficiency. These steps leverage data-driven decision-making and the lessons learned from our analysis.

1. Rural Infrastructure Development
   - Prioritizing Underserved Areas: Investments will be concentrated in rural regions where the need for water infrastructure is greatest. This includes drilling wells and installing communal taps to enhance access to safe water in remote communities.
   - Impact: By focusing on these underserved areas, we aim to reduce disparities in water access and improve the quality of life for rural populations.

2. Crime Prevention Initiatives
   - Awareness Campaigns: Conduct targeted awareness campaigns during peak crime hours and days to educate communities on safety measures.
   - Enhanced Policing: Collaborate with local authorities to improve security in high-crime areas near water sources.
   - Impact: This approach will foster safer communities and ensure uninterrupted access to water infrastructure, particularly in vulnerable regions.

3. Optimize Vendor Selection and Assignments
   - Vendor Evaluation: We identified that some teams, despite higher costs, provide exceptional value when working in challenging conditions. For example, MBS605 operates in rural Sokoto, one of the harshest regions, and their higher costs align with the demanding nature of their assignments. On the other hand, Entebbe RO Installers demonstrated cost-effectiveness by strategically minimizing travel and maximizing project completion rates.
   - Next Action: Design a vendor assignment framework that matches vendors to projects based on their strengths, geography, and cost-effectiveness. Teams will be encouraged to accept jobs closer to their base of operations to reduce travel costs and improve efficiency.

4. Implement a Centralized Project Planning System
   - Challenge Identified: Teams travelling extensively between projects contributed significantly to increased costs and delays.
   - Improvement: A centralized system will allocate projects based on proximity and vendor capabilities, reducing downtime and travel expenses. Teams will receive real-time updates on new opportunities within their vicinity, fostering quicker turnarounds and better resource utilization.
   - Impact: Initial trials show that such systems can help bring us closer to staying on budget and completing more projects in a shorter time frame.

5. Regional Cost Management
   - Focus on Rural Areas: While costs in rural Sokoto and Kilimani remain inherently high due to terrain and accessibility challenges, future budgets will more accurately reflect these realities.
   - Vendor Negotiations: Vendors will be assessed not just on cost but also on efficiency and contextual performance metrics to ensure fairness while maintaining competitiveness.

6. Team Collaboration and Best Practices Sharing
   - Case Study Learning: Entebbe RO Installers' success in minimizing costs through localized operations will be shared as a model for other vendors. Workshops will be organized to discuss strategies for reducing travel and maximizing efficiency.
   - Expected Outcome: By replicating these best practices across teams, we can collectively improve project performance, minimize delays, and align costs with our budgetary goals.

7. Continuous Monitoring and Adjustments
   - Data-Driven Decisions: Regularly monitor team performance, project costs, and vendor adherence to recommendations.
   - Feedback Loops: Collect feedback from vendors and teams to identify areas of improvement and refine strategies in real time.
   - Sustainability Check: Ensure that any cost-reduction measures do not compromise the quality of work or service delivery.
