---
title: Fundamentals of Data Engineering
author: Week 08 - sync session
...

---

#
## Assignment Review
- Review your Assignment 07
- Get ready to share

::: notes
Breakout at about 5 after the hour:
- Check in with each group 
- have students share screen
:::


## Due Friday (PR)

#
## { data-background="images/streaming-bare.svg" } 

## { data-background="images/streaming-bare-logos.jpg" } 

::: notes
- Last week we  consume messages with Spark and take a look at them
- Now, we'll transform them in spark so we can land them in hdfs
- added cloudera but that's just a distribution of hadoop
:::

# 
## Spark Stack with Kafka and HDFS

## Setup

`mkdir ~/w205/spark-with-kafka-and-hdfs`

`cd ~/w205/spark-with-kafka-and-hdfs`

```
cp ~/w205/course-content//08-Querying-Data/docker-compose.yml .
```


::: notes
Walk through the docker-compose.yml file
:::

## Spin up the cluster

```
docker-compose up -d
```
```
docker-compose logs -f kafka
```



::: notes
Now spin up the cluster
```
docker-compose up -d
```
and watch it come up
```
    docker-compose logs -f kafka
```
when this looks like it's done, you can safely detach with `Ctrl-C`.

:::


## Example: World Cup Players

## Check out Hadoop

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
::: notes
- look at tmp/ dir in hdfs and see that what we want to write isn't there already
- Can do ls options (h etc) to find out more
:::

## Should see something like:

	funwithflags:~/w205/spark-with-kafka-and-hdfs $ docker-compose exec cloudera hadoop fs -ls /tmp/
	Found 2 items
	drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
	drwx-wx-wx   - root   supergroup          0 2018-02-20 22:31 /tmp/hive

::: notes
Let's check out hdfs before we write anything to it
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
:::

## Create a topic `players`

```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic players \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists \
    --zookeeper zookeeper:32181
```

::: notes
First, create a topic `players`
```
docker-compose exec kafka kafka-topics --create --topic players --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

- Benefit, now don't have to tear this down, we'll have 2 different topics in the same kafka broker
:::

## Should show

    Created topic "players".

## Download the dataset for github players

- In `~/w205/`

```
curl -L -o players.json https://goo.gl/jSVrAe
```

::: notes
easier than github dataset b/c it's flat
:::

## Use kafkacat to produce test messages to the `players` topic

```
docker-compose exec mids \
  bash -c "cat /w205/players.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t players"
```


::: notes
```
docker-compose exec mids bash -c "cat /w205/players.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t players"
```
:::


#
## Spin up a pyspark process using the `spark` container
```
docker-compose exec spark pyspark
```

::: notes
```
docker-compose exec spark pyspark
```

- Will move to ipython shell around week 9
:::

## At the pyspark prompt, read from kafka

```
raw_players = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","players") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```

::: notes
or, without the line-conitunations,
```
raw_players = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","players").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```
:::


## Cache this to cut back on warnings later
```
raw_players.cache()
```

::: notes
- Caching this to avoid warnings bc trying to hold a persistent handle to this topic.
- Warning messages, e.g., you haven't set your state to be good with kafka etc
- What is caching?
- Lazy evaluation
  * You won't always get errors (it will be quiet) often until you do eval.
:::

## See what we got
```
raw_players.printSchema()
```

## Cast it as strings (you can totally use `INT`s if you'd like)
```
players = raw_players.select(raw_players.value.cast('string'))
```

or
```
players = raw_players.selectExpr("CAST(value AS STRING)")
```

## Write this to hdfs
```
players.write.parquet("/tmp/players")
```

::: notes
- this is writing it out to hadoop
:::

#
## Check out results (from another terminal window)

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

and
```

docker-compose exec cloudera hadoop fs -ls /tmp/players/
```

::: notes
You can see results in hadoop (from another terminal window)
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and
```
docker-compose exec cloudera hadoop fs -ls /tmp/players/
```

- Can do our h option for human readable
:::

## However (back in spark terminal window)

- What did we actually write?

```
players.show()
```

::: notes
- That's pretty ugly... let's extract the data, promote data cols to be real dataframe columns.
- We have a single column which is a strign, our jsonlines, 
- can do the take 1, take players, get value etc
- But that's kinda lame
- We want it to be easily queriable, 
- So, now let's go in and unroll the json but let's do it a dataframe at a time instead of a row at a time
:::

## Extract Data

## Deal with unicode 

```
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
```

::: notes
- If a dataset gripes about parsing `ascii` characters, you might need to default to unicode... 
- it's good practice in any case
:::


## What do we have?
- Take a look at

```
import json
players.rdd.map(lambda x: json.loads(x.value)).toDF().show()
```

::: notes
- from the dataset, I'm going to get rdd (spark's distributed dataset), apply a map to it (to work a df at a time)
- taken a df, mapped it onto an rdd, ....., convert back to a spark dataframe and now we're goig to show that
- OK,so we have our unicode errors showing up, so go back to our unicode slide, and rerun
- this is a df that looks like the content of our json and that's nice
- have to go through mapping bc you won;t have huge json file, you'll have events that are json coming from kafka etc
:::


##
```
extracted_players = players.rdd.map(lambda x: json.loads(x.value)).toDF()
```
```
from pyspark.sql import Row
extracted_players = players.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()
```
```
extracted_players.show()
```

::: notes    
- Note that first one is deprecated.  It's easier to look at though.  
- The current recommended approach to this is to explicitly create our `Row` objects from the json fields (like in 2nd example)

- Now we're going to catch that and assign it to something. 
- This is just our player data.
- ** on top of dict will return key:value pairs, but that's done in 1st row as well
:::


## Save that
```
extracted_players.write.parquet("/tmp/extracted_players")
```

::: notes
- This will be much easier to query.
- So, goal in landing this into storage is to make it efficiently querieable
- the first one wasn't bc nested
- but this is nice parquet that's flat and has nice columns
:::

## Do
```
players.show()
```

```
extracted_players.show()
```

::: notes
- Compare these 2 based on notes in last slide.
- This is great, but json is rarely flat,
- So how do wee do all this with nested dataset?
- We'll have to make some choices.
- Now we're going to work through the whole dataframe (that's what is different)
:::


#
## Example: GitHub Commits

::: notes
- Note that we didn't have to bring the cluster down bc we have 2 topics on our kafka broker now.
- I'm going to leave spark running and go over here to new container
:::

## check out hadoop

Let's check out hdfs before we write anything to it

    docker-compose exec cloudera hadoop fs -ls /tmp/

## Create a topic

```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic commits \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists \
    --zookeeper zookeeper:32181
```

::: notes
- First, create a topic `commits`
- Now actually naming our topics (ie not foo)
- Fine to have multiple topics

```
docker-compose exec kafka kafka-topics --create --topic commits --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
- which should show

    Created topic "commits".
:::

## Download the dataset for github commits
```
curl -L -o github-example-large.json https://goo.gl/Hr6erG
```

## Publish some stuff to kafka

```
docker-compose exec mids \
  bash -c "cat /w205/github-example-large.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t commits"
```

::: notes
Use kafkacat to produce test messages to the `commits` topic

```
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t commits"
```
:::

## Spin up a pyspark process using the `spark` container
```
docker-compose exec spark pyspark
```

## Read stuff from kafka

- At the pyspark prompt, read from kafka

```
raw_commits = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","commits") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```

::: notes
```
raw_commits = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

- We're selecting the value by name 
- befor we were casting, very sql like
- syntax for this in pyspark has changed greatly over time, keep an eye out for that
  * show structured streaming programming guides, @spark/apache.org
- Show can choose code in python (instead of scala etc)
:::

## Cache this to cut back on warnings
```
raw_commits.cache()
```
::: notes
- Can try it w/o caching in class to show warnings we get
:::

## See what we got
```
raw_commits.printSchema()
```

## Take the `value`s as strings
```
commits = raw_commits.select(raw_commits.value.cast('string'))
```

## Of course, we _could_ just write this to hdfs
```
commits.write.parquet("/tmp/commits")
```

- but let's extract the data a bit first...


## Extract more fields

- Let's extract our json fields again
```
extracted_commits = commits.rdd.map(lambda x: json.loads(x.value)).toDF()
```

## and see
```
extracted_commits.show()
```

::: notes
- We see it's useless because of the nesting. 
:::

## hmmm... did all of our stuff get extracted?
```
extracted_commits.printSchema()
```

- Problem: more nested json than before

::: notes
- what's going on?
- The problem is more nested json than before.
- Point of the project. Deal with this nested array. Which might change the cardinality of the dataset. (Discuss what that means)
- Here's a nice way to deal with that.
:::

## Use SparkSQL

- First, create a Spark "TempTable" (aka "View")
```
extracted_commits.registerTempTable('commits')
```

::: notes
- We'll use `SparkSQL` to let us easily pick and choose the fields we want to promote to columns.

- Note that there are other ways to extract nested data, but `SparkSQL` is the easiest way to see what's going on.
- Now we can start doing more freeform sql
:::



## Then we can create DataFrames from queries
```
spark.sql("select commit.committer.name from commits limit 10").show()
```
```
spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10").show()
```

::: notes
- 1st just mines commits
- View what you'd really like to see 
- You're transforming the data in subh a way that you're throwing things away.
- When we read from kafka with spark did the data leave kafka (ie could we just rerun it all)
- Where spark sql can really benefit you in your job.
:::


## Grab what we want
```
some_commit_info = spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10")
```

::: notes
- Compare to last week looking at this stuff using jq
::: 

## Write to hdfs

- We can write that out
```
some_commit_info.write.parquet("/tmp/some_commit_info")
```

## Check out results

-You can see results in hadoop

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and
```
docker-compose exec cloudera hadoop fs -ls /tmp/commits/
```

## Exit

- Remember, you can exit pyspark using either `ctrl-d` or `exit()`.


## Down
```
docker-compose down
```


#
## Summary

## { data-background="images/streaming-bare-logos.jpg" } 

::: notes
- What did we do in this pipeline this week
- Persistence of data in our pipeline
- We can set the retention in kafka
- currently we're using it as a buffer so keeping it for like an hour
- say we chose to do the summary, we threw a lot of the data away
- say I land that into hadoop to store long term
- this is super clean for querying, but I've lost the data
- so a lot of people leave the data in kakfa as well so they can go back and check it again
- kafka has the same kind of replicaton factors and partitions as hadoop
- gives you the choice 
:::

#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>

