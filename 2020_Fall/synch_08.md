### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #8

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2020_Fall/checklist_b4_class_assignments.md

### Discuss Project 2: Tracking User Activity

We will wait until the end of class to discuss project 2.  What we learn in class today will be needed for project 2. 

### Spark Stack with Kafka and HDFS

Setup
```
mkdir ~/w205/spark-with-kafka-and-hdfs

cd ~/w205/spark-with-kafka-and-hdfs

cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
```

Download the dataset for World Cup players in ~/w205/
```
cd ~/w205

curl -L -o players.json https://goo.gl/vsuCpZ

cd ~/w205/spark-with-kafka-and-hdfs
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

Use kafkacat to produce test messages to the players topic
```
docker-compose exec mids bash -c "cat /w205/players.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t players"
```

Spin up a pyspark process using the spark container
```
docker-compose exec spark pyspark
```

At the pyspark prompt, read from kafka
```python
raw_players = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","players").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

Cache this to cut back on warnings later
```python
raw_players.cache()
```

See what we got
```python
raw_players.printSchema()
```

Cast it as strings (you can totally use INTs if you'd like
```python
players = raw_players.select(raw_players.value.cast('string'))

OR

players = raw_players.selectExpr("CAST(value AS STRING)")
```

Write this to hdfs
```python
players.write.parquet("/tmp/players")
```

Check out results (from another terminal window)
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/players/
```

However (back in spark terminal window)
What did we actually write?
```python
players.show()
```

Extract Data

Deal with unicode
```python
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
```

What do we have?
Take a look at
```python
import json
players.rdd.map(lambda x: json.loads(x.value)).toDF().show()

extracted_players = players.rdd.map(lambda x: json.loads(x.value)).toDF()

from pyspark.sql import Row
extracted_players = players.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()

extracted_players.show()
```

Save that 
```python
extracted_players.write.parquet("/tmp/extracted_players")
```

Do
```python
players.show()

extracted_players.show()
```

Example: GitHub Commits

check out hadoop
Let's check out hdfs before we write anything to it
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/extracted_players/
```

Create a topic 
```
docker-compose exec kafka kafka-topics --create --topic commits --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Download the dataset for github commits
```
cd ~/w205

curl -L -o github-example-large.json https://goo.gl/Y4MD58

cd ~/w205/spark-with-kafka-and-hdfs
```

Publish some stuff to kafka
```
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t commits"
```

Spin up a pyspark process using the spark container (if you left pyspark running, you can skip this step):
```
docker-compose exec spark pyspark
```

Read stuff from kafka
At the pyspark prompt, read from kafka
```python
raw_commits = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

Cache this to cut back on warnings
```python
raw_commits.cache()
```

See what we got
```python
raw_commits.printSchema()
```

Take the values as strings
```python
commits = raw_commits.select(raw_commits.value.cast('string'))
```

Of course, we culd just write this to hdfs
but let's extract the data a bit first
```python
commits.write.parquet("/tmp/commits")
```

Extract more fields
Let's extract our json fields again
```python
import json

extracted_commits = commits.rdd.map(lambda x: json.loads(x.value)).toDF()
```

and see
```python
extracted_commits.show()
```

hmmm... did all of our stuff get extracted ?
Problem: more nested json than before
```python
extracted_commits.printSchema()
```

Use SparkSQL
First, create a Spark "TempTable" (aka "view")
```python
extracted_commits.registerTempTable('commits')
```

Then we can create DataFrames from queries
```python
spark.sql("select commit.committer.name from commits limit 10").show()

spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10").show()
```

Grab what we want
```python
some_commit_info = spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10")
```

Write to hdfs
We can write that out
```python
some_commit_info.write.parquet("/tmp/some_commit_info")
```

Check out results
You can see results in hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/commits/

docker-compose exec cloudera hadoop fs -ls /tmp/some_commit_info/
```

Exit
Remember you can exit pyspark using either ctrl-D or exit()
```python
exit()
```


Down
```
docker-compose down
```
