
# **Congo Conflict Analysis Project**

### **Difficulty Level: Advanced**


## **Project Overview**

I have worked on analyzing a dataset of over 17,418 conflict events that took place in The Democratic Republic of Congo from 1997-2020. 
This project involves extensive querying of number of Fatalities by location, event types, and participants, trends of number of fatalities and sexual assaults over the last years, and analyzing the main groups or parties involved in each event using Microsoft SQL Server. 
Through this project, I have tackled various SQL problems, including month-over-month percentages, creating a pivot table to see number of fatalities for each year by each group, and retrieving the percent of total fatalities by each group.
The project also focuses on research of conflict in The Congo using structured queries and functions such as window functions, date functions, aggregates, grouping, case statements, string functions and much more.




## **Solving Research Questions**

### Solutions Implemented:
/*1.	What are the top 5 locations with the most fatalities for the previous 6 years.
Display the Locations and number of fatalities for each location
*/
```sql
--Get the top 5 locations with the most fatalities within the previous 6 years
WITH Ranking AS(
SELECT location, SUM(fatalities) as num_of_fatalities,event_date, rank() OVER(ORDER BY SUM(fatalities) DESC) fatalities_ranking
FROM [70-461].[dbo].[Congo conflict data]
WHERE year>= 2014
GROUP BY location, event_date)

--select just the top 5 locations with the most fatalities
SELECT location,event_date,num_of_fatalities
FROM Ranking
WHERE fatalities_ranking <=5
```
![image](https://github.com/user-attachments/assets/dc3daddd-9fed-469e-bb28-4b2fe720f629)

/*2.    How many conflict events involved the FARDC, how many fatalities are also associated with the FARDC within the last 10 years from 2020.
Display event type, number of fataliites from that event type,  number of conflicts, overrall fatalities from all conflicts involving FARDC,
and the percentage of total of fatalities for each event type.*/
```sql
--Get the number of conflicts, fatalities, and total fatalities for all conflict types combined
WITH FARDC AS (
SELECT event_type as Conflict_Type, SUM(fatalities) as Num_of_Fatalities,
count(event_type) as Num_of_Conflict, SUM(SUM(fatalities)) OVER() as Total_fatalities_from_FARDC
FROM [70-461].[dbo].[Congo conflict data]
WHERE year >=2010 AND notes LIKE '%FARDC%'
GROUP BY event_type)

--Divide the number of fatalities for each conflict type by the total fatalities of all conflict types and multiply by 100 to get percent of total
SELECT *, concat((num_of_fatalities*100/Total_fatalities_from_FARDC), '%') as Percent_of_Total
FROM FARDC
ORDER BY Percent_of_Total DESC
```
/*3.	Make a pivot table displaying each Location with the number of fatalities for the last 5 years from 2020.
Display the associated name, the years 2015-2020, and number of fatalities as values.*/
```sql
SELECT 
*
FROM 
(
SELECT location, year, fatalities	
	   FROM [70-461].[dbo].[Congo conflict data]
	   WHERE year >= 2015
)					
Factions

PIVOT(
SUM(fatalities)
FOR year IN( [2015], [2016], [2017], [2018], [2019], [2020])
) B
```

/*
4.	Combine both actor1 and actor2 together with the word “VS” in the middle.  
Display the text with the combine columns and the notes from each event.  If one of the actors between the two columns is null write “Only One Party”.
*/
```sql
-- use concat and coalesce function to combine both columns and to return a value for null values
SELECT CONCAT(COALESCE(actor1,'Only One Party'),' ','VS',' ', COALESCE(actor2,'Only One Party') ) as Participants, notes
FROM [70-461].[dbo].[Congo conflict data]
```
/*
5.	What is the percentage of protest that ends with a fatality.  Out of all protests, what is the likelihood that it will end with at least one fatality.
Display the percentage and the text behind the percentage saying “ of Protest Will End In A Fatality”
 */
 ```sql
SELECT event_type as Conflict_Type,
SUM(CASE WHEN event_type = 'Protests' AND fatalities > 0 THEN 1 END) 'Protest With Fatality',
SUM(CASE WHEN event_type = 'Protests' AND fatalities = 0 THEN 1 END) 'Protest Without a Fatality',
CONCAT(SUM(CASE WHEN event_type = 'Protests' AND fatalities > 0 THEN 1 END)*100/ SUM(CASE WHEN event_type = 'Protests' AND fatalities = 0 THEN 1 END),'%',
' ', 'of Protests Will End In A Fatality') as
Likelihood_Protest_End_With_Fatalitiy
FROM [70-461].[dbo].[Congo conflict data]
WHERE event_type = 'Protests'
group by event_type
```
/*
6.	What group have had the most involvement in sexual assault cases against civilian women between the year 2016-2020.
Rank the location from the locations with the most sexual assaults to the location with the least.
*/
```sql
SELECT  CASE 
WHEN inter1 = 1 OR inter2 = 1 THEN 'State Forces'
	    WHEN inter1 = 2 OR inter2 = 2 THEN 'Rebel Groups'
	    WHEN inter1 = 3 OR inter2 = 3 THEN 'Political Militias'
	    WHEN inter1 = 4 OR inter2 = 4 THEN 'Identity Militias'
	    WHEN inter1 = 5 OR inter2 = 5 THEN ' Rioters'
	    WHEN inter1 = 6 OR inter2 = 6 THEN 'Protesters'
	    WHEN inter1 = 8 OR inter2 = 8 THEN 'External/Other Forces' END as Faction,
		COUNT(sub_event_type) count_of_sexual_assults
FROM [70-461].[dbo].[Congo conflict data]
WHERE sub_event_type = 'Sexual Violence' AND year BETWEEN 2016 AND 2020
GROUP BY  CASE 
WHEN inter1 = 1 OR inter2 = 1 THEN 'State Forces'
	    WHEN inter1 = 2 OR inter2 = 2 THEN 'Rebel Groups'
	    WHEN inter1 = 3 OR inter2 = 3 THEN 'Political Militias'
	    WHEN inter1 = 4 OR inter2 = 4 THEN 'Identity Militias'
	    WHEN inter1 = 5 OR inter2 = 5 THEN ' Rioters'
	    WHEN inter1 = 6 OR inter2 = 6 THEN 'Protesters'
	    WHEN inter1 = 8 OR inter2 = 8 THEN 'External/Other Forces'
		END
ORDER BY COUNT(event_type) DESC
```
/*
What is the Month-Over-Month percentage change for sexual assults from january 2020 to july 2020
Display event_type, number of sexual assults each month, and month-over-month percentage change
*/
```sql
WITH month AS(
SELECT DATENAME(month,event_date) as Month,month(event_date) Month_Num,  event_type as Conflict_type, COUNT(sub_event_type) Count_of_Sexual_Assults
FROM [70-461].[dbo].[Congo conflict data]
WHERE sub_event_type = 'Sexual Violence' AND year = 2020 AND MONTH(event_date) IN(1,2,3,4,5,6,7)
GROUP BY DATENAME(month,event_date),month(event_date), event_type),

previous_month AS(
SELECT Month_Num, Month, lag(Count_of_Sexual_Assults) OVER(ORDER BY Month_Num)Previous_Month_Count_of_Sexual_Assults,
Conflict_type, Count_of_Sexual_Assults
FROM month)

SELECT Month,Conflict_type,Count_of_Sexual_Assults,  Previous_Month_Count_of_Sexual_Assults,
ROUND(((CAST(Count_of_Sexual_Assults AS FLOAT)-Previous_Month_Count_of_Sexual_Assults)/Previous_Month_Count_of_Sexual_Assults)*100,2) Month_Over_Month_Percentage_Change
FROM previous_month
```
/*
8.   What interaction type is the most common?  How many fatalities have there been from the top interaction type in the last 10 months?
*/
```sql
WITH interactions AS(
SELECT CASE WHEN interaction =10 THEN ' State forces only'
 WHEN interaction =11 THEN ' State forces-State forces'
 WHEN interaction =12 THEN ' State forces-Rebel group'
 WHEN interaction =13 THEN ' State forces-Political militia'
 WHEN interaction =14 THEN ' State forces-Identity militia'
 WHEN interaction =15 THEN ' State forces-Rioters'
 WHEN interaction =16 THEN ' State forces-Protesters'
 WHEN interaction =17 THEN ' State forces-Civilians'
 WHEN interaction =18 THEN ' State forces-External/Other forces'
 WHEN interaction =20 THEN ' Rebel group only'
 WHEN interaction =22 THEN ' Rebel group-Rebel group'
 WHEN interaction =23 THEN ' Rebel group-Political militia'
 WHEN interaction =24 THEN ' Rebel group-Identity militia'
 WHEN interaction =25 THEN ' Rebel group-Rioters'
 WHEN interaction =26 THEN ' Rebel group-Protesters'
 WHEN interaction =27 THEN ' Rebel group-Civilians'
 WHEN interaction =28 THEN ' Rebel group-External/Other forces'
 WHEN interaction =30 THEN ' Political militia only'
 WHEN interaction =33 THEN ' Political militia-Political militia'
 WHEN interaction =34 THEN ' Political militia-Identity militia'
 WHEN interaction =35 THEN ' Political militia-Rioters'
 WHEN interaction =36 THEN ' Political militia-Protesters'
 WHEN interaction =37 THEN ' Political militia-Civilians'
 WHEN interaction =38 THEN ' Political militia-External/Other forces'
 WHEN interaction =40 THEN ' Identity militia only'
 WHEN interaction =44 THEN ' Identity militia-Identity militia'
 WHEN interaction =45 THEN ' Identity militia-Rioters'
 WHEN interaction =46 THEN ' Identity militia-Protesters'
 WHEN interaction =47 THEN ' Identity militia-Civilians'
 WHEN interaction =48 THEN ' Identity militia-External/Other forces'
 WHEN interaction =50 THEN ' Rioters only'
 WHEN interaction =55 THEN ' Rioters-Rioters'
 WHEN interaction =56 THEN ' Rioters-Protesters'
 WHEN interaction =57 THEN ' Rioters-Civilians'
 WHEN interaction =58 THEN ' Rioters-External/Other forces'
 WHEN interaction =60 THEN ' Protesters only'
 WHEN interaction =66 THEN ' Protesters-Protesters'
 WHEN interaction =67 THEN ' Protesters-Civilians'
 WHEN interaction =68 THEN ' Protesters-External/Other forces'
 WHEN interaction =70 THEN ' Civilians only'
 WHEN interaction =77 THEN ' Civilians-Civilians'
 WHEN interaction =78 THEN ' External/Other forces-Civilians'
 WHEN interaction =80 THEN ' External/Other forces only'
 WHEN interaction =88 THEN ' External/Other forces-External/Other forces' END as interactions,
 SUM(fatalities) Sum_of_Fatalities
FROM [70-461].[dbo].[Congo conflict data]
GROUP BY CASE WHEN interaction =10 THEN ' State forces only'
 WHEN interaction =11 THEN ' State forces-State forces'
 WHEN interaction =12 THEN ' State forces-Rebel group'
 WHEN interaction =13 THEN ' State forces-Political militia'
 WHEN interaction =14 THEN ' State forces-Identity militia'
 WHEN interaction =15 THEN ' State forces-Rioters'
 WHEN interaction =16 THEN ' State forces-Protesters'
 WHEN interaction =17 THEN ' State forces-Civilians'
 WHEN interaction =18 THEN ' State forces-External/Other forces'
 WHEN interaction =20 THEN ' Rebel group only'
 WHEN interaction =22 THEN ' Rebel group-Rebel group'
 WHEN interaction =23 THEN ' Rebel group-Political militia'
 WHEN interaction =24 THEN ' Rebel group-Identity militia'
 WHEN interaction =25 THEN ' Rebel group-Rioters'
 WHEN interaction =26 THEN ' Rebel group-Protesters'
 WHEN interaction =27 THEN ' Rebel group-Civilians'
 WHEN interaction =28 THEN ' Rebel group-External/Other forces'
 WHEN interaction =30 THEN ' Political militia only'
 WHEN interaction =33 THEN ' Political militia-Political militia'
 WHEN interaction =34 THEN ' Political militia-Identity militia'
 WHEN interaction =35 THEN ' Political militia-Rioters'
 WHEN interaction =36 THEN ' Political militia-Protesters'
 WHEN interaction =37 THEN ' Political militia-Civilians'
 WHEN interaction =38 THEN ' Political militia-External/Other forces'
 WHEN interaction =40 THEN ' Identity militia only'
 WHEN interaction =44 THEN ' Identity militia-Identity militia'
 WHEN interaction =45 THEN ' Identity militia-Rioters'
 WHEN interaction =46 THEN ' Identity militia-Protesters'
 WHEN interaction =47 THEN ' Identity militia-Civilians'
 WHEN interaction =48 THEN ' Identity militia-External/Other forces'
 WHEN interaction =50 THEN ' Rioters only'
 WHEN interaction =55 THEN ' Rioters-Rioters'
 WHEN interaction =56 THEN ' Rioters-Protesters'
 WHEN interaction =57 THEN ' Rioters-Civilians'
 WHEN interaction =58 THEN ' Rioters-External/Other forces'
 WHEN interaction =60 THEN ' Protesters only'
 WHEN interaction =66 THEN ' Protesters-Protesters'
 WHEN interaction =67 THEN ' Protesters-Civilians'
 WHEN interaction =68 THEN ' Protesters-External/Other forces'
 WHEN interaction =70 THEN ' Civilians only'
 WHEN interaction =77 THEN ' Civilians-Civilians'
 WHEN interaction =78 THEN ' External/Other forces-Civilians'
 WHEN interaction =80 THEN ' External/Other forces only'
 WHEN interaction =88 THEN ' External/Other forces-External/Other forces' END ),

 Ranking AS(
 SELECT *, RANK() OVER(ORDER BY Sum_of_Fatalities DESC) Ranking
 FROM interactions)

 SELECT interactions, Sum_of_Fatalities
 FROM Ranking
 WHERE Ranking <=10
```

 /*
 9.	 What is the running total of number of fatalities for each month for each event type within the last 6 months of 2020.
 */
```sql
 WITH months AS (
 SELECT event_type as Conflict_Type, DATENAME(MONTH,event_date) as month_name,month(event_date) as month, SUM(fatalities) Num_of_Fatalities
 FROM [70-461].[dbo].[Congo conflict data]
WHERE event_date BETWEEN DATEADD(MONTH,-6, (SELECT MAX(event_date) FROM [70-461].[dbo].[Congo conflict data])) AND 
(SELECT MAX(event_date) FROM [70-461].[dbo].[Congo conflict data])
GROUP BY event_type, DATENAME(MONTH,event_date), month(event_date))

SELECT Conflict_Type, month_name, Num_of_Fatalities,SUM(Num_of_Fatalities) OVER(PARTITION BY Conflict_Type ORDER BY month ) as Running_Total
FROM months
```

## **Learning Outcomes**

This project enabled me to:
- Use critical thinking to retrieve data for research purposes.
- Use advanced SQL techniques, including window functions, subqueries, joins, case statements, date functions, and more.
- Conduct in-depth Research analysis using SQL Server.
- Optimize query performance and handle large datasets efficiently.

---

## **Conclusion**

This advanced SQL project successfully demonstrates my ability to solve real-world problems using structured queries. From analyzing locations in The Congo that had the most fatalities to analyzing recent trends in number of sexual assaults within the recent years, the project provides valuable insights into national challenges.

By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive help learn about national issues within The Congo.


