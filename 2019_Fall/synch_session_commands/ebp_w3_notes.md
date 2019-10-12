### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #3

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Summer/synch_session_commands/checklist_b4_class_assignments.md

#### First three queries of part 1 of project 1

What's the size of this dataset? (i.e., how many trips)
```sql
#standardSQL
SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
983648
```

What is the earliest start time and latest end time for a trip?
```sql
#standardSQL
SELECT min(start_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`

#standardSQL
SELECT max(end_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
2013-08-29 09:08:00
2016-08-31 23:48:00
```



How many bikes are there?
```sql
#standardSQL
SELECT count(distinct bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
700
```

#### Using some Linux command line utilities to process a csv (comma separated value) file

The website explainshell.com is really good at finding out about linux command line:
https://explainshell.com/

Save data into your `w205` directory:
```
cd ~/w205
curl -L -o annot_fpid.json https://goo.gl/qWiu7d
curl -L -o lp_data.csv https://goo.gl/FDFPYB
```

Install jq
```
sudo apt update
sudo apt install jq
```

What's in this file?
```
head lp_data.csv
tail lp_data.csv
```

What are variable in here?
```
head -n1 lp_data.csv
```

How many entries?
```
cat lp_data.csv | wc -l
```

How about sorting?
```
cat lp_data.csv | sort
```

Take a look at what options there are for sort
```
man sort
```

fix so sorting correctly

```
cat lp_data.csv | sort -g
cat lp_data.csv | sort -n
```

Find out which topics are more popular

What have we got in this file?
```
head annot_fpid.json
```

Hmmm, what now? jq
pretty print the json
```
cat annot_fpid.json | jq .
```

Just the terms
```
cat annot_fpid.json | jq '.[][]'
```

Remove the ""s
```
cat annot_fpid.json | jq '.[][]' -r
```

Can we sort that?
```
cat annot_fpid.json | jq '.[][]' -r | sort 
```

Hmmm, there's lots of repeated terms
Unique values only
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq 
```

How could I find out how many of each of those unique values there are?
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c 
```

Now, how could I sort by that?
Ascending
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -g
```
Descending
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -gr
```

So, what are the top ten terms?
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -gr | head -10
```



#### Google BigQuery command line interface: bq cli

Please double check your VM and make sure that you have the option "Allow full access to all Cloud APIs" is set.

To check this: Navigation Menu => Compute Engine => VM Instances => see the list of VMs => click on the name of your VM to bring up the "VM instance details" page => scroll down to the end where you will see the section "Cloud API access scopes" and make sure it says "Allow full access to all Cloud APIs"

If it does not: if the VM is running, probably best to stop it => scroll to the top of the "VM instance details" page => click "EDIT" => scroll down to change "Cloud API access scopes" to "Allow full access to all Cloud APIs" => click "Save" button => start the VM

See if bq runs:
```
bq
```

Run some Google BigQuery queries using the bq cli
```
bq query --use_legacy_sql=false 'SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer: something like 107,501,619


How many stations are there?
```
bq query --use_legacy_sql=false 'SELECT count(distinct station_id) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer: something like 75

How long a time period do these data cover?
```
bq query --use_legacy_sql=false 'SELECT min(time), max(time) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer:  2013-08-29 12:06:01.000 UTC  2016-08-31 23:58:59.000 UTC   

#### Advanced options 

Sort by 'product_name'
```
cat lp_data.csv | awk -F',' '{ print $2,$1 }' | sort
```

Fix the ""s issue
```
cat lp_data.csv  | awk -F',' '{ print $2,$1 }' | sed 's/"//' | sort | less
```

#### Jupyter Notebooks for Project 1

Clone down your project 1 repo.  Open Jupyter Hub.  Navigate and open the Jupyter Notebook for project 1.

The queries will not run as is.  Replace them with the following queries:

```
! bq query --use_legacy_sql=FALSE 'SELECT trip_id, start_station_name, end_station_name, count(trip_id) as trip_freq FROM `bigquery-public-data.san_francisco.bikeshare_trips` GROUP BY trip_id, start_station_name, end_station_name ORDER BY trip_freq DESC LIMIT 5'
```

```
! bq query --use_legacy_sql=FALSE --format=csv 'SELECT trip_id, start_station_name, end_station_name, count(trip_id) as trip_freq FROM `bigquery-public-data.san_francisco.bikeshare_trips` GROUP BY trip_id, start_station_name, end_station_name ORDER BY trip_freq DESC LIMIT 5' > result.csv
```

#### Running Google BigQuery queries from the command line using bq may encounter some issues with the ' and ` characters

BigQuery table names need to be wrapped in back ticks (single back quotes) ` 

When running queries from the command line, the examples we have wrap them with single ticks (single straight quotes) '
as in the following example

```
bq query --use_legacy_sql=false 'SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```

What if we need to have a single quote inside?  There are several ways to get around this. One is to wrap the query in double single quotes " and escape the back ticks with a back slash as in \`
as in the following example

```
bq query --use_legacy_sql=false "SELECT count(*) FROM \`bigquery-public-data.san_francisco.bikeshare_trips\` where start_station_name = 'Mezes' "
```

If you are using an editor on the desktop to save queries while you are developing them, please be careful as some editors will place an artistic interpretation of the characters.  They will often slant single or double quotes to make them look better.  If it does this, the changes to the characters won't work in BigQuery.

