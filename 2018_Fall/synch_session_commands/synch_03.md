### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #3

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Ownership issues between science and root

Files created in your droplet will be owned by science with group science. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **droplet** when logged in as science to change the owner to science and the group to science, recursively, for a directory:
```
sudo chown -R science:science w205
```

### First three queries of assignment 02

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

### Using some Linux command line utilities to process a csv (comma separated value) file

The website explainshell.com is really good at finding out about linux command line:
https://explainshell.com/

Save data into your `w205` directory:
```
cd ~/w205
curl -L -o annot_fpid.json https://goo.gl/rcickz
curl -L -o lp_data.csv https://goo.gl/rks6h3
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

(Up until now, we have been using the droplet. The jq command is not part of standard linux.  jq is installed in our docker container midsw205/base, so we can startup a container and use it to run the following commands:

```


::: notes

- It will scroll all over the page, 
- b/c it's json, it's just one line, so head is everything
:::

## Hmmm, what now? jq

pretty print the json


`cat annot_fpid.json | jq .`

::: notes

- There's a slide for a jq reference at the end here, we're just going to do a little in class
:::

## Just the terms

`cat annot_fpid.json | jq '.[][]'`

::: notes

- 
:::

## Remove the ""s

`cat annot_fpid.json | jq '.[][]' -r`

::: notes

- 
:::

## Can we sort that?

`cat annot_fpid.json | jq '.[][]' -r | sort `

::: notes

- Hmmm, there's lots of repeated terms
:::

## Unique values only

    cat annot_fpid.json | jq '.[][]' -r | 
    sort | uniq 

::: notes

- 
:::

## How could I find out how many of each of those unique values there are?

    cat annot_fpid.json | jq '.[][]' -r | 
    sort | uniq -c 

::: notes

- 
:::

## Now, how could I sort by that?


    cat annot_fpid.json | jq '.[][]' -r | 
    sort | uniq -c | sort -g

Ascending

    cat annot_fpid.json | jq '.[][]' -r | 
    sort | uniq -c | sort -gr

Descending

::: notes

- 
:::

## So, what are the top ten terms?

    cat annot_fpid.json | jq '.[][]' -r | 
    sort | uniq -c | sort -gr | head -10


::: notes

- 
:::

#

## bq cli

## setup

(from your mids droplet)

- auth the GCP client
  ```
  gcloud init
  ```
  and copy/paste the link

- associate `bq` with a project
  ```
  bq
  ```
  and select project if asked

::: notes
`gcloud init` will print an oauth link that needs to be copied over to a browser
:::

##

```
bq query --use_legacy_sql=false '
SELECT count(*)
FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```

::: notes
107,501,619

The point: you can use `select *` to actually answer questions.

```
bq query --use_legacy_sql=false 'SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
:::

## How many stations are there?

##

```
bq query --use_legacy_sql=false '
SELECT count(distinct station_id)
FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```

::: notes
The point: how to count unique
Answer: something like 75

```
bq query --use_legacy_sql=false 'SELECT count(distinct station_id) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
:::


## How long a time period do these data cover?

##

```
bq query --use_legacy_sql=false '
SELECT min(time), max(time)
FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```


::: notes
- 2013-08-29 12:06:01.000 UTC   
- 2016-08-31 23:58:59.000 UTC   

```
bq query --use_legacy_sql=false 'SELECT min(time), max(time) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
:::


#
## Generate Ideas

- What do you know?
- What will you need to find out?

::: notes

- breakout
- Generate Ideas = get them going on generating questions for project 
- If they don't come up with anything, ask:
  1. What do you know?
    * i.e., what variables do you have? what do they mean? 
  2. What will you need to find out?
    * i.e., how to use those variables in some combo to figure out:
    * What's a trip?
    * What's a commuter trip?
    * etc
:::

#
## Summary
- Command line tools and jq to dive into your data
- BigQuery from the command line


#
## Extras

::: notes
- All of this is stuff you can use or not.
:::

## Resources

## sed and awk

<http://www.catonmat.net/blog/awk-one-liners-explained-part-one/>
<http://www.catonmat.net/blog/sed-one-liners-explained-part-one/>

## jq

<https://stedolan.github.io/jq/tutorial/>

## Advanced options 

## Sort by 'product_name'

```
cat lp_data.csv | awk -F',' '{ print $2,$1 }' | sort
```

::: notes
```
cat lp_data.csv | awk -F',' '{ print $2,$1 }' | sort
```

- Put in extras for add ons or activities if folks finish early

- This switches the columns and sorts on LP title
- but you find out that some LPs have ""s around the titles
:::


## Fix the ""s issue

```
cat lp_data.csv  | awk -F',' '{ print $2,$1 }' | sed 's/"//' | sort | less
```

::: notes
```
cat lp_data.csv  | awk -F',' '{ print $2,$1 }' | sed 's/"//' | sort | less
```

- the sed part here takes out the "" 
- and then we sort based on title
:::




#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>
