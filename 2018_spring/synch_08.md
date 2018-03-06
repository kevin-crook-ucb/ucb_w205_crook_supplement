# Kevin Crook's week 8 synchronous session supplemental notes

Overview of today's synch session:

* Before class in your droplet:
  * get rid of any old docker containers, unless you know you are running something:
    * docker rm -f $(docker ps -aq)
  * update course-content:
    * docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
    * cd ~/course-content
    * git pull --all
    * exit
  * update the following docker images: 
    * docker pull confluentinc/cp-zookeeper:latest
    * docker pull confluentinc/cp-kafka:latest
    * docker pull midsw205/cdh-minimal:latest
    * docker pull midsw205/spark-python:0.0.5
    * docker pull midsw205/base:latest
* Activity 1:
  * Purpose: add hadoop to our docker cluster to store json data of world cup players, seeing problems with unicode characters, encoding unicode with utf-8 to fix those problems, writing to parquet format in the hadoop hdfs (hadoop distributed file system)
  * create a docker cluster with 5 containers: zookeeper, kafka, cloudera (hadoop distribution), spark, and mids
  * access the hadoop hdfs
  * create a kafka topic called players
  * download a json file of world cup players
  * publish the json data of world cup player to the kafka topic players
  * using spark consume the json data from the kafka topic
  * write the json data to parquet format in hdfs
  * see that the json data has unicode characters
  * encode the unicode characters with utf-8 to fix the issue
  * write the json data in utf-8 to hdfs
* Activity 2:
  * Purpose: similar the activity 1, use but use the github dataset, see that the github dataset is nested and not flat like the world cup dataset was, use spark methods to handle the nested structure, including spark sql queries and deriving data frames from spark sql queries, save to parquet format in hdfs
  * we will use the running cluster above without terminating it
  * create a kafka topic called commits
  * download the json file of github commits data
  * publish the json file of github commits data to the kafka topic commits
  * using spark consume the json data from the kafka topic
  * write the json data to parquet format in hdfs
  * see that the json data is nested
  * use spark sql to handle the nested data
  * extract part of the data using spark sql and write that to parquet format in hdfs
  
## Activity 1

Create a directory for spark with kafka and hadoop hadfs
```
mkdir ~/w205/spark-with-kafka-and-hdfs
cd ~/w205/spark-with-kafka-and-hdfs
```

Copy the yml file:
```
cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
```

We will go through the yml file in class.  You will need to edit the file and change the volume mapping as we have done before.

Start the docker cluser
```
docker-compose up -d
```

As we did before, you may want to use the following command to watch kafka come up. Multiple command line windows work best for this.  Remember to use control-C to exit.
```
docker-compose logs -f kafka
```

The hadoop hdfs is a separate file system from our local linux file system.  The directories are different and have different paths.  You will need to use the following command to view a directory listing of the hdfs directory /tmp.  If the cluster is first coming up, or if you have recently added a directory or file, it may take time to show up.  Remember that big data architectures are eventually consistent and not immediately consistent.
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

The output should look similar to this.  Remember it may take a while to show consistency:
```
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2018-02-20 22:31 /tmp/hive
```

Create a kafka topic called players.  This is the same procedure we have used several times before with the foo topic:
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

The same command on 1 line for convenience:
```
docker-compose exec kafka kafka-topics --create --topic players --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

You should see the following or similar:
```
Created topic "players".
```

Download the dataset for world cup players in json format.  Remember our downloads go into the ~/w205 directory:
```
cd ~/w205/
curl -L -o players.json https://goo.gl/jSVrAe
cd ~/w205/spark-with-kafka-and-hdfs
```

Use kafkacat to publish the world cup players json data to the kafka topic players:
```
docker-compose exec mids \
  bash -c "cat /w205/players.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t players"
```

The same command on 1 line for convenience:
```
docker-compose exec mids bash -c "cat /w205/players.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t players"
```

Start a spark pyspark shell in the spark container.  Remember that pyspark is the python interface to spark.
```
docker-compose exec spark pyspark
```

Write python code in pyspark to consume from the kafka topic:
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

The same command on 1 line for convenience:
```
raw_players = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","players").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

The following command will cache the spark data structure.  Without it every time something isn't in memory it will generate a warning message.  The warning messages don't hurt anything, but are very distracting.  This is due to spark's use of "lazy evaluation" which is a hallmark of big data architecture.  Add that to our big data architecture we have seen so far: immutable, eventually consistent, lazy evaluation.
```
raw_players.cache()
```

Before you print the schema, what do you expect to see?  It's the same shema we have seen when consuming a kafka topic in the past:
```
raw_players.printSchema()
```

Remember that the value attribute of a kafka schema will be stored as raw bytes which is not easily human readable.  So we convert it to strings so humans can read it:
```
players = raw_players.select(raw_players.value.cast('string'))
```

An alternative way to the previous command (you only need to do one):
```
players = raw_players.selectExpr("CAST(value AS STRING)")
```

Write the players data frame to a parquet file in hadoop hdfs.  Note that this is writing to hadoop hdfs and not to the local linux file system - big difference.  hdfs is intended to have a virtual presence on all nodes in the hadoop cluster.  Parquet format is a binary format for storing data in binary format in a file in columnar format.  Parquet files are immutable.  Remember that immutable files are a  hallmark of big data architecture.  It allows them to be pushed out in a hadoop cluster, stored in object store (which we could use an elastic query resource against), delivered in content delivery networks, etc.
```
players.write.parquet("/tmp/players")
```

Using another command line window, (keep pyspark running), use the following hdfs command to see the directory and files we just created. Another hallmark of big data architecture is that when we write, we usually specify a directory rather than a file. In this case see the players is a directory and not a file. Note the unique naming convention of the data files.  In our case we only see 1 file.  But if we had multiple nodes, each node would have it's own file.  This is so they can write in parallel.  We can do this due the "shared nothing architecture" which is another hallmark of big data architecture.  Also note that files will have a maximum size of 2 GiB.  This is so they can be written using 32 bit pointers rather than 64 bit pointers.  
```
docker-compose exec cloudera hadoop fs -ls /tmp/
docker-compose exec cloudera hadoop fs -ls /tmp/players/
```

Going back to our pyspark, let's look at the data we wrote.  We may see some unicode characters.  Most western alphabets need only 7 bits to store all characters, so we can store it in 1 byte (8 bits).  (We used to call it ASCII).  However, other languages of the world have their own alphabets and need 2 bytes to store a single character.   Unicode is the standard for this, but unicode has the disadvantage of storing 2 bytes for every character whether it needs it or not.  As a compromise, utf-8 format was invented.  This format will use only 1 byte if the character needs only 1 byte, and 2 bytes if the character needs 2 bytes.  It can do this by using the extra bit as a signal.
```
players.show()
```

The following code will set standard output to write data using utf-8 instead of unicode.  In the modern era, it's almost always a good idea to always use utf-8:
```
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
```

Take a look at the players data formatted for json:
```
import json
players.rdd.map(lambda x: json.loads(x.value)).toDF().show()
```

Create a new data frame to hold the data in json format:
```
extracted_players = players.rdd.map(lambda x: json.loads(x.value)).toDF()
```

(krc - stopped reformatting here)
As an alternative to the previous command, we 
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

