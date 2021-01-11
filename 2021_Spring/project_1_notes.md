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
ORDER BY duration_sec ASC
```

The durations are given in terms of seconds.  It might be more useful to see them in terms of minutes or maybe even in hours.

```sql
#standardSQL
SELECT duration_sec, 
       CAST(ROUND(duration_sec / 60.0) AS INT64) AS duration_minutes,
       CAST(ROUND(duration_sec / 3600.0) AS INT64) AS duration_hours_rounded,
       ROUND(duration_sec / 3600.0, 1) AS duration_hours_tenths
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
ORDER BY duration_sec ASC
```

You may want to do some exploratory queries to see if durations are reasonable.  Try some queries to see how many, what percentage are below a reasonable threshold, or above a reasonable threshold.

#### Start and end dates use an inappropriate data type

You will notice that the start_date and end_date use an inappropriate data type.  They use the TIMESTAMP which is intended to store system level time stamps in UTC, not application level data with time zone offsets.  However, they do store the dates and times in Pacific time so you can assume the times are all Pacific. They also adjust for daylight savings times, so you don't need to make any adjustments.  

This is just like the proverbial real world - people use the wrong data types all the time!

#### Adding some columns to make the day of week and hour easier to analyze

The basic query would be to just pull out the start_date as it is stored:

```sql
#standardSQL
SELECT start_date 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
ORDER BY start_date ASC
```

In order to make our analytics easier, we may want to do some "feature engineering", in the form of creating some new columns that are easier to work with:

```sql
#standardSQL
SELECT start_date,
       EXTRACT(DAYOFWEEK FROM start_date) AS dow_int,
       CASE EXTRACT(DAYOFWEEK FROM start_date)
           WHEN 1 THEN "Sunday"
           WHEN 2 THEN "Monday"
           WHEN 3 THEN "Tuesday"
           WHEN 4 THEN "Wednesday"
           WHEN 5 THEN "Thursday"
           WHEN 6 THEN "Friday"
           WHEN 7 THEN "Saturday"
           END AS dow_str,
       CASE 
           WHEN EXTRACT(DAYOFWEEK FROM start_date) IN (1, 7) THEN "Weekend"
           ELSE "Weekday"
           END AS dow_weekday,
       EXTRACT(HOUR FROM start_date) AS start_hour,
       CASE 
           WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
           WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
           WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
           END AS start_hour_str
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
ORDER BY start_date ASC
```

#### Summary queries which aggregate data can be very useful in analytics

This queries aggregates trips that have different start and end stations.  After the aggregation, it filters only those with more than 100 trips.

```sql
#standardSQL
SELECT start_station_name, end_station_name, count(*) AS number_of_trips
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE start_station_name <> end_station_name
GROUP BY start_station_name, end_station_name
HAVING number_of_trips > 100
ORDER BY 1, 2, 3
```
