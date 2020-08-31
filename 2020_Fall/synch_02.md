### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #2

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2020_Fall/checklist_b4_class_assignments.md

#### Project 1: Query Project

#### Google BigQuery Queries

Suggestions:
* use the Chrome browser
* use an incognito window to avoid cookie conflicts
* make you have given them a credit card and enabled billing on the project

Links:

https://cloud.google.com/bigquery/

https://cloud.google.com/bigquery/docs/

https://cloud.google.com/bigquery/public-data/

There are 2 sets of tables related to the San Francisco Bike Share: a static set and a dynamic set.  The dynamic set is the one mentioned in link above.  Since it's much easier for beginners to use the static set, that is what we will use.  If you use the dynamic set, the query results will not match the examples we give you, and can possible change with every 15 minute update.  Here are the tables we will be using:

```
bigquery-public-data.san_francisco.bikeshare_status
bigquery-public-data.san_francisco.bikeshare_stations
bigquery-public-data.san_francisco.bikeshare_trips
```

Example first query:
```sql
#standardSQL
SELECT *
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many events are there?
```sql
#standardSQL
SELECT count(*)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many stations are there?
```sql
#standardSQL
SELECT count(distinct station_id)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How long a time period do these data cover?
```sql
#standardSQL
SELECT min(time), max(time)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many bikes does station 90 have (hint: total bikes should be docks_available + bikes_available)?

Does this query give us the answer?
```sql
#standardSQL
SELECT station_id, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
WHERE station_id = 90
```

No, it's time dependent.  So, let's try this query:
```sql
#standardSQL
SELECT station_id, docks_available, bikes_available, time, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
WHERE station_id = 90
ORDER BY total_bikes
```

## Create our own private dataset named bike_trip_data, create our own private table named total_bikes in our private dataset, run some queries against our private table

In the Google BigQuery user interface, on the left side panel, you will see the name of your project.  To the right of the project name, you will see a dropdown arrow.  Click on the dropdown arrow and choose "Create new dataset" and use this to create a new dataset named bike_trip_data.

Execute the following query.  Once the results come back, towards the top right of the results panel, choose "Save as Table".  Create a table named total_bikes in and put it in the dataset you just created.

```sql
#standardSQL
SELECT station_id, docks_available, bikes_available, time, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

Using the GUI examine the new table you created going through all of the tabs.  Pay close attention to the naming and use it to create a similar queries to the ones below.

```sql
#standardSQL
SELECT distinct (station_id), total_bikes
FROM `xxxx.bike_trip_data.total_bikes`
```

```sql
#standardSQL
SELECT distinct station_id, total_bikes
FROM `xxxx.bike_trip_data.total_bikes`
WHERE station_id = 22
```
 
 
