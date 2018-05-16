---
title: Fundamentals of Data Engineering
author: Week 02 - sync session
...

---

# 

## Assignment 1
- We will usually do a breakout to share solutions and ask questions, but this week's was pretty straightforward.
- Questions on process?

## Your droplet set up

- repos cloned:
    - `course-content`
    - `assignment-01-<user-name>`

## How to do a PR

- Review process from gui 

::: notes
we defer cli-support for this until next week
:::

## Due tomorrow morning


# 
## Some things about this class

## How to read

- Two screens (or devices)
- Reading
- Following along with coding

## Pacing

- What you can do
- What you can understand

::: notes
We'll be reading/watching screencasts ahead of what we can do
:::


# 
## Where are we in the pipeline

![](images/pipeline-overall.svg)

## Events

- What sort of events feed this pipeline?
- How were these events captured?

::: notes
Fed from device events:

- station kiosk (?)
- user app (?)
:::

## Main thing to pay attention to

- Pipeline is provided for this example
- We're _using_ it to answer business questions

::: notes
How would we build this ourselves?
:::


# 
## Big Ideas

## Business Decisions

- All about the business

- Data-Driven Business Decisions ...are queries

::: notes
in order for business decisions to be based on data, you have to be able to interact with the data.

business requirements get encoded as queries of data
:::

## Translation

- SQL queries are really pretty easy
- How to get to the queries from the questions, sometimes not so much


# 
## Query Project

- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.

- We will be using the Bay Area Bike Share Trips Data (<https://cloud.google.com/bigquery/public-data/bay-bike-share>). 

::: notes
Go over what the dataset is on the webpage
:::

## Problem Statement
- You're a data scientist at Ford GoBike (<https://www.fordgobike.com/>), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership. 

## Questions

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


# 
## Get Going: Google account

- Go to <https://cloud.google.com/bigquery/>
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, 

::: notes
(**OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO**)
:::


## Working with BQ gui

<https://bigquery.cloud.google.com/table/bigquery-public-data:san_francisco.bikeshare_status>

## Tutorial

<https://www.w3schools.com/sql/default.asp>


#
## Some annoying specific stuff about BQ

## the `;`

    SELECT * 
    FROM Customers;

VS 

    SELECT * 
    FROM Customers

::: notes
- Nearly all other sql implementations (and what students will see in the tutorial), end statements with a `;`
- BQ doesn't
- btw, the CAPITALIZATION isn't necessary :)
:::

## Legacy vs Standard SQL


    SELECT *
    FROM [bigquery-public-data:san_francisco.bikeshare_trips]

VS 

    #standardSQL
    SELECT * 
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`

::: notes
- Google was going to create their own sql that worked different :)
- It's silly, but the way the table reference part, the "FROM" part, is written is different.
- So, now there's the `#standardSQL` flag
:::

## The Big Difference


    SELECT distinct(bikes_available) 
    FROM [bigquery-public-data:san_francisco.bikeshare_status]


NO


    #standardSQL
    SELECT distinct(bikes_available) 
    FROM `bigquery-public-data.san_francisco.bikeshare_status`

YES

::: notes
- It's in doing things with distinct that I've noticed the biggest differences from regular (aka "standard" SQL)
- You get: Error: syntax error at: 1.1 - 1.42. SELECT DISTINCT is currently not supported. Please use GROUP BY instead to get the same effect.
:::

## For this class

	#standardSQL
	SELECT * 
	FROM `bigquery-public-data.san_francisco.bikeshare_status`


- More similar to command line bq
- More like most other SQL implementations

::: notes
We're doing this one, but you can use either
:::


#
## Querying Data

## How many events are there?

::: notes
For the following slides,
Wait on the question slide for a minute to give everyone a chance to try it.

Then reveal the next slide with query.

- Optional: you can do these in groups and ask people to 

  * report out
  * share issues, false starts they ran into 

[Obviously more useful on more complicated queries]
:::

##

	#standardSQL
	SELECT count(*)
	FROM `bigquery-public-data.san_francisco.bikeshare_status`


::: notes
107,501,619

The point: you can use `select *` to actually answer questions.
:::

## How many stations are there?

##

	#standardSQL
	SELECT count(distinct station_id)
	FROM `bigquery-public-data.san_francisco.bikeshare_status`

::: notes
The point: how to count unique
Answer: something like 75
:::


## How long a time period do these data cover?

##

	#standardSQL
	SELECT min(time), max(time)
	FROM `bigquery-public-data.san_francisco.bikeshare_status`


::: notes
- 2013-08-29 12:06:01.000 UTC	
- 2016-08-31 23:58:59.000 UTC	
:::

## How many bikes does station 90 have?


::: notes
Break into groups here.
Give them a few minutes, have someone from each group screen share and present their query.
If they don't tell you, ask why they made the choices they made.
I decided that a station's total bikes would `= docks_available + bikes_available`.
:::


##

	#standardSQL
	SELECT station_id, 
	(docks_available + bikes_available) as total_bikes
	FROM `bigquery-public-data.san_francisco.bikeshare_status`
	WHERE station_id = 90

::: notes
Stuff to explore:

- each station "has" different unique numbers of bikes [probably b/c e.g., docks are added to stations, etc, etc, etc]

**Next slides will help unpack what they find here**
:::

## What's up with that?

::: notes
Getitng into queries to help figure out the issue from last slide
:::

##

	#standardSQL
	SELECT station_id, docks_available, bikes_available, time, 
	(docks_available + bikes_available) as total_bikes
	FROM `bigquery-public-data.san_francisco.bikeshare_status`
	WHERE station_id = 90
    ORDER BY total_bikes


::: notes
The point: 

- query returns 8916 results 
- but if ordered by `total_bikes`, can click "First" and "Last" to see what the values are
:::

## Get a table with `total_bikes` in it

::: notes
"Ok, so we don't want to go clicking through 8900 results to figure out what the unique values for `total_bikes` for a station are."

- On this one, just show it
:::

## 

	#standardSQL
	SELECT station_id, docks_available, bikes_available, time, 
	(docks_available + bikes_available) as total_bikes
	FROM `bigquery-public-data.san_francisco.bikeshare_status`

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


