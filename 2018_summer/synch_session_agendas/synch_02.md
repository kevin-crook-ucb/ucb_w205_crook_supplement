# UCB MIDS W205 Summer 2018 - Kevin Crook's agenda for Synchronous Session #2

## As always, remember to update the course-content repo in your docker container in your droplet

In your droplet, startup a container with volume mapping:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```
Using the bash shell running inside the container, change to the course content directory:
```
cd ~/course-content
```
Update your course-content repo:
```
git pull --all
```
Exit the docker container:
```
exit or control-D
```

## Discuss the Query Project

Involves assignments 2, 3, 4, and 5

Warning: each assignment has the same header.  The header applies to all of 2, 3, 4, and 5.  Do only the detailed in the assignment part below the header.

## SQL Tutorial

<https://www.w3schools.com/sql/default.asp>

## Signup for Google Cloud account 

<https://cloud.google.com>

Everyone should be signed up for the Google Cloud account before class as mentioned in slack.

Some tips:
* Use the Chrome browser.  Google makes Google Cloud.  Google makes Chrome.  Google tests Google Cloud using Chrome.  Other browsers will hit things that won't work correctly.
* You may have multiple Google accounts. Check and switch if necessary to the right account by clicking on the person icon in the upper right corner.
* You must currently be on a project with billing enabled, otherwise something will work and others that require billing will not.
* If you attempt to use the bike share dataset in BigQuery and it won't come up, it's probably one of the above.

## Link to the google bigquery bike share dataset

<https://bigquery.cloud.google.com/table/bigquery-public-data:san_francisco.bikeshare_status>

## Legacy vs Standard SQL

We will be using Standard SQL instead of the Legacy SQL.  It will be required for assignments and points will be deducted if it's not used.  You **must** include the #standardSQL directive both in the query when running and also in assignments or points will be deducted, unless it's covered in other syntax.

Be careful if you are pulling code off the internet, as a lot of it may be specific to Legacy SQL.  

Standard SQL supports more constructs and has more accurate results (is accuracy required for Big Data Analytics?)

Example of Legacy SQL (do not use)
```sql
SELECT *
FROM [bigquery-public-data:san_francisco.bikeshare_trips]
```

Same example in Standard SQL (use)
```sql
#standardSQL
SELECT * 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Standard SQL supports more constructs such as distinct():
```sql
#standardSQL
SELECT distinct(bikes_available) 
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

## Let's write and run some example queries against the bikeshare_status table

We will go to breakout and let each breakout group attempt to answer each question below.  Scroll down to see the answers.

1. Select all column, all rows from the bikeshare_status table.

2. How many events are there? (hint: each row in the bikeshare_status table is considered an event)

3. How many stations are there? (hint: there is a station_id column in the bikeshare_status table, but remember each station_id may have more than 1 event.  We only want to count each station once)

4. How long a time period do these data cover? (hint: there is a time column in the bikeshare_status table.  Find the earliest time and 
the latest time for all times)

5. How many bikes does station 90 have (hint: total bikes should be docks_available + bikes_available. In this model, each bike has a dock and if the dock is empty, it means the bike is in use.  Does the number of bikes at station 90 change over time?)


## Queries which answer the previous questions

1. Select all columns, all rows from the bikeshare_status table:
```sql
#standardSQL
SELECT * 
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

2. How many events are there?
```sql
#standardSQL
SELECT count(*)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

3. How many stations are there?
```sql
#standardSQL
SELECT count(distinct station_id)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

4. How long a time period do these data cover?
```sql
#standardSQL
SELECT min(time), max(time)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

5. How many bikes does station 90 have (hint: total bikes should be docks_available + bikes_available)?

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



::: notes
- This is the query to create the `total_bikes` table (which is totally a view, but BQ is weird about views, something about legacy sql vs standard sql)
- Do "Save Table"
- Window will pop up, need to have added a dataset to your project earlier, then enter dataset name and add a name for the table.
- I'm calling it `total_bikes`
:::

## 

	#standardSQL
	SELECT distinct (station_id), total_bikes
	 FROM `ambient-cubist-185918.bike_trips_data.total_bikes`

::: notes
This shows that you get multiple entries for each `station_id` b/c diff values of total bikes
:::

##

	#standardSQL
	SELECT distinct station_id, total_bikes
	FROM `ambient-cubist-185918.bike_trips_data.total_bikes`
	WHERE station_id = 22

::: notes
This lets you explore each station's total number of bikes
:::



## Independent Queries

<https://www.w3schools.com/sql/default.asp>

::: notes
If there's any time, break in groups to do whatever questions they come up with. 
Rotate between groups to see what folks are coming up with.
:::


# 
## SecureShell (SSH)

#
## remote terminal connections

##

    ssh science@xxx.xxx.xxx.xxx

::: notes
for your cloud instance, look up:
- the ip address
- password for the `science` user
:::


#
## copying files

##

On your laptop, run

    scp some_file science@xxx.xxx.xxx.xxx:

or 

    scp some_file science@xxx.xxx.xxx.xxx:/tmp/


::: notes
copying files from your laptop to the instance

note the colon!
:::

##

On your laptop, run

    scp science@xxx.xxx.xxx.xxx:~/w205/a_file.py .


::: notes
copying files from the instance to your laptop

note the period!
:::


# 
## Summary

- Business questions
- Answered using empirical data
- By running queries against (raw?) events
- Need a pipeline in place to capture these raw events
- SSH

#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>


