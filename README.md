# database_project2_results

See results of the following queries based on the log associated with the query number:

1) Number of cases per county, showing the cases, county and state names
select max(Cases.`Cases`) as `Cases`, C.`Name` as `County_Name`, S.`Name` as `State_Name`  from `County_Date_Cases` as Cases left join `County` as C on Cases.`County_ID` = C.`County_ID`  left join `State` as S on S.`State_ID` = C.`State_ID` group by C.`County_ID`;

2) Number of deaths per county, showing the deaths, county and state names
select max(Cases.`Deaths`) as `Deaths`, C.`Name` as `County_Name`, S.`Name` as `State_Name` from `County_Date_Cases` as Cases left join `County` as C on Cases.`County_ID` = C.`County_ID`  left join `State` as S on S.`State_ID` = C.`State_ID` group by C.`County_ID`;

3) Number of deaths per state, showing the deaths and state names
select max(Cases.`Deaths`) as `Deaths`, S.`Name` as `State_Name` from `County_Date_Cases` as Cases left join `County` as C on Cases.`County_ID` = C.`County_ID` left join `State` as S on S.`State_ID` = C.`State_ID` group by C.`State_ID`;

4) Number of counties per state, number of cases, and average number of cases per county, ordered by descending number of counties
select (COUNT(*)/30) as `Number of Counties` , max(CDC.`Cases`) as `Max Cases`,   max(CDC.`Cases`)/(COUNT(*)/30) as `Average`, S.`Name` as `State_Name` from `County_Date_Cases` as CDC left join `County` as C on CDC.`County_ID` = C.`County_ID` left join `State` as S on S.`State_ID` = C.`State_ID` group by C.`State_ID`order by `Number of Counties` desc;

5) Number of cases per county in one state, showing the cases and county names
select max(Cases.`Cases`) as `Cases`, C.`Name` as `County_Name` from `County_Date_Cases` as Cases left join `County` as C on Cases.`County_ID` = C.`County_ID` left join `State` as S on S.`State_ID` = C.`State_ID` where S.`Name` like "Maryland%" group by C.`County_ID`;

6) Sum of total number of deaths across all counties 
select sum(totaldeaths)  from ( select max(`Deaths`) as totaldeaths, `County_ID` from `County_Date_Cases` as Cases group by `County_ID` ) t ;

7) Number of cases and deaths for a given county (DC) in the last 30 days
select `Date`, `Cases`, `Deaths`  from `County_Date_Cases` where`County_ID` like "11001%" ;

8) Mask use by county
select Name, Never_Mask, Rarely_Mask, Sometimes_Mask, Frequently_Mask, Always_Mask  from `County` order by `Always_Mask` desc;

9) All colleges with county and college IDs
Select `County_ID`, `College_ID`, `Name`  From `College`;

10) All hospitals in a given city
select C.`Name` , H.`Name` , H.`Hospital_ID`, H.`Ownership`, H.`Emergency_Services`, H.`Type`  from `Hospital` as H join `City` as C on H.`City_ID` = C.`City_ID` where C.`Name` like "Washington%";

11) All cities that don’t have a hospital with emergency services
Create view citiesnoemergency asc select a.`City_ID`, GROUP_CONCAT(a.`Emergency_Services`) from `Hospital` a  left outer join ( select `City_ID`, COUNT(*) from `Hospital` where `Emergency_Services` like "Yes%" group by  `City_ID` ) b on a.`City_ID` = b.`City_ID` where b.`City_ID` is null group by `City_ID`;

Create view countiessnoemergency as select c.`City_ID`, c.`Name`, c.`County_ID` from `City` as c join citiesnoemergency where c.`City_ID` = citiesnoemergency.`City_ID`;

12) Number of cases and deaths for counties that have cities without any hospitals having emergency services
select cdc.`County_ID`, max(cdc.`Cases`) as `Cases`, max(cdc.`Deaths`) as `Deaths`, cno.`Name` as `County N ame`, cno.`City_ID` from `County_Date_Cases` as cdc join countiesnoemergency as cno where cdc.`County_ID` = cno.`County_ID` group by cdc.`County_ID`;

13) All counties that have a college in it
create view countieswcolleges as select distinct `County_ID` from `College` inner join `County` using(`County_ID`);

14) All counties that don’t have a college in it
create view countieswcolleges as select co.`County_ID`from `County` cowhere co.`County_ID` not in (select coll.`County_ID` from countieswcolleges as coll);

15) Average number of cases for counties that do have a college
select avg(totalcases) from (select max(cdc.`Cases`) as totalcases from `County_Date_Cases` as cdc  join countieswcolleges cwc where cdc.`County_ID` = cwc.`County_ID` group by cdc.`County_ID` ) t;

16) Average number of cases for counties that don’t have a college
select avg(totalcases) from ( select max(cdc.`Cases`) as totalcases from `County_Date_Cases` as cdc join countieswocolleges cwoc  where cdc.`County_ID` = cwoc.`County_ID` group by cdc.`County_ID` ) t;

17) Counties having (never_mask + rarely_mask) < 0.1
select c.`County_ID`, c.`Name`, c.`Never_Mask`, c.`Rarely_Mask` from `County` as c having (c.`Never_Mask` + c.`Rarely_Mask`) < 0.1;

18) Deaths and cases by county with always_mask > 0.75: 
select cdc.`County_ID`, max(cdc.`Cases`) as `Cases`, max(cdc.`Deaths`) as `Deaths`, c.`Always_Mask` from `County` as c join `County_Date_Cases` as cdc where cdc.`County_ID` = c.`County_ID` group by cdc.`County_ID` having (c.`Always_Mask` > 0.75 order by max(cdc.`Cases`) desc;

19) Average cases by county with always_mask > 0.75
select avg(totalcases) from (  select cdc.`County_ID`, max(cdc.`Cases`) as totalcases, c.`Always_Mask` from `County` as c  join `County_Date_Cases` as cdc  where cdc.`County_ID` = c.`County_ID` group by cdc.`County_ID` having (c.`Always_Mask` > 0.75) ) t;

20) Average cases by county with always_mask < 0.25:
select avg(totalcases)  from ( select cdc.`County_ID`, max(cdc.`Cases`) as totalcases, c.`Always_Mask`  from `County` as c  join `County_Date_Cases` as cdc  where cdc.`County_ID` = c.`County_ID` group by cdc.`County_ID`  having (c.`Always_Mask` < 0.25)  ) t;
