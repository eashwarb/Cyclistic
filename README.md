# Introduction
Cyclistic is an imaginary ride sharing company based in Chicago. Until today, the company believed and worked on building general awarness about ride sharing and attracting customers from different consumer segments. Key to that was the single ride and full day passes. These are called casual riders. There is another category of riders called annual riders who pay for annual memeberships. According to the Head of the Marketing, these annual memberships are the key for the company's future growth. Finance Analysts also confirm the same by giving numbers that show that annual members are more profitable than the casual riders for the company.The marketing team believes that, instead of creating campaigns to draw all-new customers to purchase the annual memberships,there is a pretty good chance of converting the casual members into annual riders.But for that we have to understand how these riders actually take rides and their usage patterns.

## Business Problem
How do annual members and casual riders use Cyclistic bikes differently?

## Approach
This analysis will be conducted in 4 stages which will help in coming with a sound analysis.

1) ASK 
2) Prepare 
3) Process 
4) Analyze 
5) Share

## Ask
While the bigger question to answer is regarding the usage patterns, we will have to ask many small questions to answer the ultimate question correctly. This is important for solving any business problem because we don't want to kick start a project with out knowing the boundaries and key aspects of the project. So the questions we are asking here include -

1) What is the problem we are trying to address? 
2) What is the motive of the project? 
3) Who are all the stake holders in the project? 
4) How will my insights help the company?

Now that we have answers for the question, lets get to the next stage.

## Prepare
This stage is when we collect the data, verify the credibility and sources of the data. It is also important to address the data integrity over the course of the project.

This, being a capstone project, we don't have to worry about most of them. We have access to 12 months of ride data, spread across 4 million rows and 15 columns, is available to dowload from the website in CSV format.

### Data Source

Motivate International Inc.

## Process
Tools chosen for analysis - SQL and Tableau

All the 12 csv files containing the ride data have been imported into a MySQL database -'Cyclistic'. While going through the data, it was found that station IDs were integers untilNovember 2020 and post that, the data type changed to varchar. So clearly, the formats of station IDs have changed starting from Dec ’20. To negate this, all the station IDs were converted to varchar.

Starting off with a query to create a table to house all the 12 months data.

```
create table cyclistic_uncleaned (
ride_id varchar(100),
rideable_type varchar(100),
started_at varchar(100),
ended_at varchar(100),
start_station_name varchar(100),
end_station_name varchar(100),
member_casual varchar(100));
```
Populating the newly created table with all the data.

```
insert into cyclistic_uncleaned
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual from 202006_divvy_tripdata_csv
union all 
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202007_divvy_tripdata_csv
union all 
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202008_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202009_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202010_divvy_tripdata_csv
union all 
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202011_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202012_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202101_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202102_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202103_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202104_divvy_tripdata_csv
union all
select ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, member_casual  from 202105_divvy_tripdata_csv;
```
Check for the empty/null cells in any of the columns

```select * from cyclistic_uncleaned where(start_station_name ='' or end_station_name='');

delete from cyclistic_uncleaned where(start_station_name ='' or end_station_name='');
```
Changing the datatypes for 'started_at' and 'ended_at' columns so that they can be used as time stamps

```
update cyclistic_uncleaned
set started_at =str_to_date(started_at, '%Y-%m-%d %H:%i:%s');
update cyclistic_uncleaned
set ended_at =str_to_date(ended_at, '%Y-%m-%d %H:%i:%s');
```
Checking for anomalies in data
```
select * from cyclistic_uncleaned where ended_at<started_at; 
```
9606 rows have such data which is clearly some sort of error

```
delete from cyclistic_uncleaned where ended_at<started_at; 
```
removed all the negative ride length data

 ```
 select * from cyclistic_uncleaned where ended_at=started_at; 
 ```
#3104 rows have such data
```
delete from cyclistic_uncleaned where ended_at=started_at; 
```
removed zero ride time data

```
select timestampdiff(second, started_at, ended_at) from cyclistic_uncleaned where timestampdiff(second, started_at, ended_at)<60;
```
check for ridetimes less than 60s. 44206 rows of data with less than a minute ridetimes

```
delete from cyclistic_uncleaned where timestampdiff(second, started_at, ended_at) <60;
```

Creating a new sheet to populate it with the cleaned data
 ```
 create table cyclistic_final 
select 
ride_id,
rideable_type,
started_at,
timestampdiff(minute, started_at, ended_at) as ride_time,
weekday(started_at) as ride_day, 
start_station_name,
end_station_name,
member_casual
from cyclistic_uncleaned;
```
Creating new columns for the ride days and ride months

```
Alter table cyclistic_final
modify ride_day varchar(16);

update cyclistic_final
set ride_day = 
case when ride_day = 0 then 'Monday'
when ride_day = 1 then 'Tuesday'
when ride_day = 2 then 'Wednesday'
when ride_day = 3 then 'Thursday'
when ride_day = 4 then 'Friday'
when ride_day = 5 then 'Saturday'
when ride_day = 6 then 'Sunday'
else null  end;
```

Creating a new table and adding month names instead of numbers
```
create table cyclistic_done 
select 
ride_id,
rideable_type,
started_at,
ride_time,
ride_day, 
extract(month from started_at) as ride_month,
start_station_name,
member_casual
from cyclistic_final;

Alter table cyclistic_done
modify ride_month varchar(5);

update cyclistic_done
set ride_month = 
case when ride_month = 1 then 'Jan'
when ride_month= 2 then 'Feb'
when ride_month= 3 then 'Mar'
when ride_month = 4 then 'Apr'
when ride_month = 5 then 'May'
when ride_month = 6 then 'Jun'
when ride_month = 7 then 'Jul'
when ride_month = 8 then 'Aug'
when ride_month = 9 then 'Sep'
when ride_month = 10 then 'Oct'
when ride_month = 11 then 'Nov'
when ride_month = 12 then 'Dec'
else null  end;
```
        #We have a cleandata set to work with.
## Analyze
Now we look at some interesting insights derived from the cleaned data set. To start off, lets look at the number of rides taken by casual and annual members over the past 12 months.<br/>
            <img width="540" alt="download x" src="https://user-images.githubusercontent.com/79919319/131563298-db6c5ca1-97f7-403b-93f4-a993d66504e3.png"><br/>
As expected, the annual member rides are more than the casual rides and this is by a margin of 38%.

This chart gives an idea about the number of rides taken by the two types of customers in each month.
Note - Our data begins from June 2020 - May 2021.<br/>
            <img width="540" alt="download 2" src="https://user-images.githubusercontent.com/79919319/131562781-7c3d4814-f6b8-446a-81aa-8d3351a23868.png"><br/>
It is evident from the chart that riders from both the groups tend to have seasonal impact on bike usage.

Similarly let us have a look at the average trip durations on monthly basis. <br/>
           <img width="540" alt="download 3" src="https://user-images.githubusercontent.com/79919319/131562821-62432515-297a-4808-8e0a-8a9a8401d5ac.png"> <br/>

This graph also shows the seasonal impact of ride lengths on both the consumer types. Also we notice stark difference in the ride lengths in casual riders and members. The ride lengths are pretty consistent in the case of annual members, while the ride times rise in the summer in the case of casual riders.

To draw some day wise insights regarding the bike usage we have similar charts plotted for each day of the week. <br/>
          <img width="540" alt="download 4" src="https://user-images.githubusercontent.com/79919319/131562888-14f5ae1a-d8a9-44bb-99c4-5720b1e50272.png"> <br/>
          <img width="540" alt="download 5" src="https://user-images.githubusercontent.com/79919319/131563005-de4e7cc3-c9b5-4208-b997-0935141889dd.png"> <br/>


It is clearly evident from these graphs that annual members seem to have most of their usage on weekdays only and we also notice that ride times are consistent through out the weekdays and slightly increase during the weekends.

## Share
At the final step of the project, let us present some insights we have derived from our analysis and make some data driven recommendations to the Marketing Team of the company.

### Insights
1) It is fairly clear that, the annual riders are mostly weekday riders and based on their consistent ride times it is evident that these people are restricting their usage to offices, schools and colleges.The average ride times in the weekdays stand at 14.2 mins and weekend averages stand at 16.9 mins.

2) Contrary to the annual riders, casual riders are mostly weekend riders with the average ride times going up by 17% in weekends and more importantly massive 47% difference in avg weekend and avg weekday rides.

3) Both the 'average ride time' and 'number of rides' parameters across both the groups, nosedived in the month of november and continued the trend in december and january. Only possible explaination could be the onset of winter in US.

4) Summer breaks in colleges and schools help in increased tourist activity and accordingly we see the number casual riders peak in the period of June, July and August. Average ride times in the case of casual riders over 55 minutes in the months of June and July compared to the rest of the year average of 36.4 minutes, that is a 52% increase. Similar trend is seen in the case of number of rides taken, with the highest number of rides recorded in the month August for both the customer groups.

### Recommendations
Based on the insisghts, following recommendations are made to the marketing team of Cyclistic.

1) Marketing plans to attract annual memberships should be planned way ahead of the summer and are to be excuted just before the summer to gain maximum traction.

2) Considering the fact that most of the annual members are daily commuters, offering special discounts to school/college/office going people might help the company attract more riders from that segment. To compensate for the discount we can limit such passes to only weekday usage.

3) Implementing leaderboards for the riders can help generate intereset among the public. Incentives like Monthly rewards or reward points for riders on the leaderboards can help the company draw new set of customers.
