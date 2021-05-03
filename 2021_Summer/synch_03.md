### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #3

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2021_Spring/checklist_b4_class_assignments.md

#### First three queries of part 1 of project 1

What's the size of this dataset? (i.e., how many trips)
```sql
SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
983648
```

What is the earliest start time and latest end time for a trip?

You can do it as two separate queries:

```sql
SELECT min(start_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`

SELECT max(end_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
2013-08-29 09:08:00
2016-08-31 23:48:00
```

Or do it as 1 query:

```sql
SELECT min(start_date), max(end_date)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

How many bikes are there?
```sql
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

The latest release of Google AI Notebooks appears to have jq installed.  Prior releases did not have it installed. You can test this by running the following command and seeing if it prints the help for the command:
```
jq
```

If the jq command above doe not print the help for the command, you can install it as follows.  (If it printed the help for the command, you do not need to install it).
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


Verify that bq runs (it should respond with a version and details of the command line arguments available):
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

#### Running Google BigQuery queries from the command line using bq may encounter some issues with the ' and ` characters

BigQuery table names need to be wrapped in back ticks (single back quotes) ` 

When running queries from the command line, the examples we have wrap them with single ticks (single straight quotes) '

What if we need to have a single quote inside?  There are several ways to get around this. One is to wrap the query in double single quotes " and escape the back ticks with a back slash as in \`
as in the following example

```
bq query --use_legacy_sql=false "SELECT count(*) FROM \`bigquery-public-data.san_francisco.bikeshare_trips\` where start_station_name = 'Mezes' "
```

If you are using an editor on the desktop to save queries while you are developing them, please be careful as some editors will place an artistic interpretation of the characters.  They will often slant single or double quotes to make them look better.  If it does this, the changes to the characters won't work in BigQuery.
