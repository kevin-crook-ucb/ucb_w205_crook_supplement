### Project 1 Notes

First, be sure you are using the right tables:
```
bigquery-public-data.san_francisco.bikeshare_status
bigquery-public-data.san_francisco.bikeshare_stations
bigquery-public-data.san_francisco.bikeshare_trips
```

#### Best to create your own dataset with your own views

I think it's best to create your own dataset with views.  The views allow you to have your own virtual copy of the data with dirty data fixed or removed, only the columns that you need for your analysis, any new columns that you have derived and added, summary views with aggregated data, etc.  Also having easy column names instead of complicated expressions makes it a whole lot easier.

#### trip duration times

Let's consider trip duration times. Run the query below and see the first few pages and last few.  What do you notice?

```sql
#standardSQL
SELECT duration_sec
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by duration_sec asc
```

The durations are given in terms of seconds.  It might be more useful to see them in terms of minutes or maybe even in hours.

```sql
#standardSQL
SELECT duration_sec, 
       cast(round(duration_sec / 60.0) as INT64) as duration_minutes,
       cast(round(duration_sec / 3600.0) as INT64) as duration_hours_rounded,
       round(duration_sec / 3600.0, 1) as duration_hours_tenths
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by duration_sec asc
```

You may want to do some exploratory queries to see if durations are reasonable.  Try some queries to see how many, what percentage are below a reasonable threshold, or above a reasonable threshold.

#### Start and end dates use an inappropriate data type

You will notice that the start_date and end_date use an inappropriate data type.  They use the TIMESTAMP which is intended to store system level time stamps in UTC, not application level data with time zone offsets.  However, they do store the dates and times in Pacific time so you can assume the times are all Pacific. They also adjust for daylight savings times, so you don't need to make any adjustments.  

This is just like the proverbial real world - people use the wrong data types all the time!

The basic query would be to just pull out the start_date as it is stored:

```sql
#standardSQL
SELECT start_date 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by start_date asc
```

In order to make our analytics easier, we may want to do some "feature engineering", in the form of creating some new columns that are easier to work with:

```sql
#standardSQL
SELECT start_date,
       extract(dayofweek from start_date) as dow_int,
       case extract(dayofweek from start_date)
           when 1 then "Sunday"
           when 2 then "Monday"
           when 3 then "Tuesday"
           when 4 then "Wednesday"
           when 5 then "Thursday"
           when 6 then "Friday"
           when 7 then "Saturday"
       end as dow_str,
       case 
           when extract(dayofweek from start_date) in (1, 7) then "Weekend"
           else "Weekday"
       end as dow_weekday,
       extract(hour from start_date) as start_hour
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by start_date asc
```
