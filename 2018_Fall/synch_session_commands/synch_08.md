# work in progress, please wait until this message is removed to use this, until I've had a chance to work through it myself

### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #8

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md

### Discuss Project 2: Tracking User Activity

Assignment 6 - Get and Clean Data 

Assignment 7 - Setup Pipeline

Assignment 8 - Build and Write-up Pipeline

### Spark Stack with Kafka and HDFS

Setup
```
mkdir ~/w205/spark-with-kafka-and-hdfs

cd ~/w205/spark-with-kafka-and-hdfs

cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
```

Spin up the cluster
```
docker-compose up -d

docker-compose logs -f kafka
```
Example: World Cup Players

Check out Hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

Should see something like:
```
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2018-02-20 22:31 /tmp/hive
```

Create a topic players
```
docker-compose exec kafka kafka-topics --create --topic players --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should show
```
Created topic "players".
```

Download the dataset for github players in ~/w205/
```
cd ~/w205

curl -L -o players.json https://goo.gl/vsuCpZ

cd ~/w205/spark-with-kafka-and-hdfs
```

Use kafkacat to produce test messages to the players topic
```
docker-compose exec mids bash -c "cat /w205/spark-with-kafka-and-hdfs/players.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t players"
```

Spin up a pyspark process using the spark container
```
docker-compose exec spark pyspark
```

At the pyspark prompt, read from kafka
```
raw_players = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","players").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

Cache this to cut back on warnings later
```
raw_players.cache()
```

See what we got
```
raw_players.printSchema()
```

Cast it as strings (you can totally use INTs if you'd like
```
players = raw_players.select(raw_players.value.cast('string'))

OR

players = raw_players.selectExpr("CAST(value AS STRING)")
```

Write this to hdfs
```
players.write.parquet("/tmp/players")
```

Check out results (from another terminal window)
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/players/
```

However (back in spark terminal window)
What did we actually write?
```
players.show()
```

Extract Data

Deal with unicode
```
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
```

What do we have?
Take a look at
```
import json
players.rdd.map(lambda x: json.loads(x.value)).toDF().show()

extracted_players = players.rdd.map(lambda x: json.loads(x.value)).toDF()

from pyspark.sql import Row
extracted_players = players.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()

extracted_players.show()
```

Save that 
```
extracted_players.write.parquet("/tmp/extracted_players")
```

Do
```
players.show()

extracted_players.show()
```

Example: GitHub Commits

check out hadoop
Let's check out hdfs before we write anything to it
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

Create a topic 
```
docker-compose exec kafka kafka-topics --create --topic commits --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Download the dataset for github commits
```
cd ~/w205

curl -L -o github-example-large.json https://goo.gl/Y4MD58

TBD
```

Using kafkacat publish the github json dataset to the topic commits
```
docker-compose exec mids \
  bash -c "cat /w205/spark-with-kafka-and-hdfs/github-example-large.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t commits"
```

Same command on 1 line for convenience:
```
docker-compose exec mids bash -c "cat /w205/spark-with-kafka-and-hdfs/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t commits"
```

Using our pyspark command prompt, consume from the kafka topic commits into a kafka data frame:
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

Same command on 1 line for convenience:
```
raw_commits = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

As before we will cache to supress warning messages which are distracting:
```
raw_commits.cache()
```

What should the schema for raw_commits look like?  Remember it came from a kafka topic.  Use the following command to find out:
```
raw_commits.printSchema()
```

Remember that the value from a kafka topic will be a raw byte string, which as before, we will convert into a string:
```
commits = raw_commits.select(raw_commits.value.cast('string'))
```

The following command will wthe commits data fram to a parquet file in hdfs:
```
commits.write.parquet("/tmp/commits")
```

As before, let's extract our json fields:
```
extracted_commits = commits.rdd.map(lambda x: json.loads(x.value)).toDF()
```

Show the data frame after the json extraction:
```
extracted_commits.show()
```

Notice that this time we have nested json data.  Before our json data was flat.

Let's print the schema:
```
extracted_commits.printSchema()
```

We will now use spark sql to deal with the nested json.

First, cretae a spark temporary table called commits based on the data frame.  registerTempTable() is a method of the spark class data frame.
```
extracted_commits.registerTempTable('commits')
```

Issue spark sql against the temporary table commits that we just registered:
```
spark.sql("select commit.committer.name from commits limit 10").show()
spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10").show()
```

Save the results of the query into another data frame:
```
some_commit_info = spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10")
```

Write the data frame holding the results of our query to a parquet file in hdfs:
```
some_commit_info.write.parquet("/tmp/some_commit_info")
```

Go to our other linux command line window and use the following command to see the directory and files in hdfs:
```
docker-compose exec cloudera hadoop fs -ls /tmp/
docker-compose exec cloudera hadoop fs -ls /tmp/commits/
```

Go to the pyspark window and exit pyspark:
```
exit()
```


Tear down the docker cluster and make sure it's down:
```
docker-compose down
docker-compose ps
```

