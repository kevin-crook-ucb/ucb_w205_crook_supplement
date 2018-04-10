# under construction, please wait

# Kevin Crook's week 13 synchronous session supplemental notes

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
    * docker pull midsw205/hadoop:0.0.2
      * (instead of midsw205/cdh-minimal:latest)
    * docker pull midsw205/spark-python:0.0.6
      * (up from 0.0.5)
    * docker pull midsw205/presto:0.0.1
      * (new image)
    * docker pull midsw205/base:0.1.9
* SSH
  * Let's finish up SSH from last week.
* Activity
  * Previously in Project 3:
    * We have built a docker cluster with zookeeper, kafka, cloudera hadoop, spark w python, and mids containers.  
    * We have designed and run a web API server, generated API calls using curl on the command line, a real web browser, telnet, and PuTTY raw mode.  
    * We have created a kafka topic and written API events and supporting web logs to the kafka topic.  
    * We have used spark to read the kafka topic and filter, flatten, transform, etc. events using massively parallel processing methods.  
    * We have used spark in the pyspark python oriented command line, using the spark-submit job submission style interface, and using jupyter notebook.  
    * We have also written our spark data frames using the massively parallel processing methods out to parquet format in hadoop hdfs.  We saw overwrite errors and handled them by deleting the directory and its contents or by using another name.  We also changed our code to use the overwrite option.  
    * We have used spark SQL to query our data frame using the convenience of SQL instead of the transforms.
    * We have used kafkacat in an interactive mode where it will show events as they come through. 
    * We have used Apache Bench to automate stress testing of our web API.  
    * We saw that multiple schema types on the kafka topic would break our code, so we beefed up our code to process different schemas differently.
    * We read in the data frames that we saved out to parquet format in hdfs.  (massively parallel read) 
    * We copied a spark data frame into a Pandas data frame for more convenient processing of results.
    * For spark, we have used the interfaces: pyspark command line, Jupyter Notebook, and spark-submit
  * This week:
    * We will introduce Hive which is an SQL based data warehouse platform that runs on top of hadoop hdfs.  
    * We will use hive to create a schema on read for our parquet files stored in hdfs.
    * We will run some queries from hive against our parquet files stored in hdfs.
    * We will use python code to see another way to interact with the hive metastore.
    * We will introduce another tool, Presto, to do ad hoc queries of our parquet files in hadoop hdfs.
    * (At this point we have basically built the architecture that we used with the query project for the bike share data in the Google Big Query)
    * We will introduce Spark Streaming.  So far we have just read the entire topic on kafka and written everything to hdfs.  Now we want to define batches of 10 second intervals to write in parquet to hdfs.  We will see how immutability plays into this.

## Activity


---
title: Fundamentals of Data Engineering
author: Week 13 - sync session
...

---

#
## Get Started
```
git pull in ~/w205/course-content
mkdir ~/w205/full-stack2/
cd ~/w205/full-stack2
cp ~/w205/course-content/13-Understanding-Data/docker-compose.yml .
docker-compose pull
cp ~/w205/course-content/13-Understanding-Data/*.py .

```

::: notes
:::




#


## { data-background="images/pipeline-steel-thread-for-mobile-app.svg" } 

::: notes
Let's walk through this
- user interacts with mobile app
- mobile app makes API calls to web services
- API server handles requests:
    - handles actual business requirements (e.g., process purchase)
    - logs events to kafka
- spark then:
    - pulls events from kafka
    - filters/flattens/transforms events
    - writes them to storage
- presto then queries those events
:::


# 
## Flask-Kafka-Spark-Hadoop-Presto Part II
::: notes
- last week we did spark from files
- ended with spark files reading from kafka, did some munging events, extracted events, json explode, did some filtering for event types.
:::

#
## Setup

## The `docker-compose.yml` 

Create a `docker-compose.yml` with the following
```
---
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    expose:
      - "2181"
      - "2888"
      - "32181"
      - "3888"
    extra_hosts:
      - "moby:127.0.0.1"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    expose:
      - "9092"
      - "29092"
    extra_hosts:
      - "moby:127.0.0.1"

  cloudera:
    image: midsw205/hadoop:0.0.2
    hostname: cloudera
    expose:
      - "8020" # nn
      - "8888" # hue
      - "9083" # hive thrift
      - "10000" # hive jdbc
      - "50070" # nn http
    ports:
      - "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"

  spark:
    image: midsw205/spark-python:0.0.6
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    expose:
      - "8888"
    #ports:
    #  - "8888:8888"
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
      HIVE_THRIFTSERVER: cloudera:9083
    extra_hosts:
      - "moby:127.0.0.1"
    command: bash

  presto:
    image: midsw205/presto:0.0.1
    hostname: presto
    volumes:
      - ~/w205:/w205
    expose:
      - "8080"
    environment:
      HIVE_THRIFTSERVER: cloudera:9083
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:0.1.9
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    expose:
      - "5000"
    ports:
      - "5000:5000"
    extra_hosts:
      - "moby:127.0.0.1"
```

::: notes
k, z same
- cloudera a little thinner, a few extra environment variables
- presto new conainer

:::

## Spin up the cluster

```
docker-compose up -d
```

::: notes
- Now spin up the cluster
```
docker-compose up -d
```
- Notice we didn't actually create a topic as the broker does this for you
:::

## Web-app

- Take our instrumented web-app from before
`~/w205/full-stack/game_api.py`

```python
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask, request

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    event.update(request.headers)
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "This is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "Sword Purchased!\n"
```

::: notes
- same web app as before
:::

## run flask
```
docker-compose exec mids \
  env FLASK_APP=/w205/full-stack/game_api.py \
  flask run --host 0.0.0.0
```

::: notes

```
docker-compose exec mids env FLASK_APP=/w205/full-stack/game_api.py flask run --host 0.0.0.0
```

:::

## Set up to watch kafka

```
docker-compose exec mids \
  kafkacat -C -b kafka:29092 -t events -o beginning
```


::: notes
- new terminal window, leave up
- running kafkacat without -e so it will run continuously

```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```
:::

## Apache Bench to generate data

```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/purchase_a_sword
```

::: notes
- Will do lots more events with streaming later in class.

```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
```
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
```
```
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```
:::

# 

## Some Spark to Write Events

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))

    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()

    extracted_purchase_events \
        .write \
        .mode('overwrite') \
        .parquet('/tmp/purchases')


if __name__ == "__main__":
    main()
```
::: notes
- Reminder of pieces and parts
- Filtering on is_purchase
- Write to tmp/purchases in hdfs
:::
## Run this

```
docker-compose exec spark spark-submit /w205/full-stack2/filtered_writes.py
```


## See purchases in hdfs

```
docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```




#
## Queries from Presto

## Hive metastore

- Track schema
- Create a table

::: notes
- The Hive metastore is a really common tool used to keep track of schema for
tables used throughout the Hadoop and Spark ecosystem (schema registry).

- To "expose" the schema for our "purchases"... we need to create a table in the
hive metastore.

- There are two ways
  * Run hive explicitly and create an external table
  * Run spark, create a 

:::

## Hard Way


```
docker-compose exec cloudera hive
```


::: notes
- Run hive in the hadoop container
- This is what you would do, don't need to actually do it, skip to easier way
:::

## 

```sql
create external table if not exists default.purchases2 (
    Accept string,
    Host string,
    User_Agent string,
    event_type string,
    timestamp string
  )
  stored as parquet 
  location '/tmp/purchases'
  tblproperties ("parquet.compress"="SNAPPY");
```

::: notes
```
create external table if not exists default.purchases2 (Accept string, Host string, User_Agent string, event_type string, timestamp string) stored as parquet location '/tmp/purchases'  tblproperties ("parquet.compress"="SNAPPY");
```


:::

## Or... we can do this an easier way


```
docker-compose exec spark pyspark
```


::: notes
- run spark
:::


##

```python
df = spark.read.parquet('/tmp/purchases')
df.registerTempTable('purchases')
query = """
create external table purchase_events
  stored as parquet
  location '/tmp/purchase_events'
  as
  select * from purchases
"""
spark.sql(query)
```

::: notes
```
spark.sql("create external table purchase_events stored as parquet location '/tmp/purchase_events' as select * from purchases")
```
:::

## Can just include in job

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .enableHiveSupport() \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))

    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()

    extracted_purchase_events.registerTempTable("extracted_purchase_events")

    spark.sql("""
        create external table purchases
        stored as parquet
        location '/tmp/purchases'
        as
        select * from extracted_purchase_events
    """)


if __name__ == "__main__":
    main()
```

::: notes
- Modified filtered_writes.py to register a temp table and then run it from w/in spark itself
:::

## Run this

```
docker-compose exec spark spark-submit /w205/full-stack2/write_hive_table.py
```

## See it wrote to hdfs

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
::: notes
- This is the first spark job to run - it does it all, read, flatten, write, ?query
:::
## and now ...

- Query this with presto

```
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```

::: notes
- Presto just a query engine
- it's talking to the hive thrift server to get the table we just added
- connected to hdfs to get the data
- Querying with presto instead of spark bc presto scales well, handles a wider range of sql syntax, can start treating like a database, can configure it to talk to cassandra, s3 directly, kafka directly, mysql, good front end for your company's data lake

:::

## What tables do we have in Presto?

```
presto:default> show tables;
   Table   
-----------
 purchases 
(1 row)

Query 20180404_224746_00009_zsma3, FINISHED, 1 node
Splits: 2 total, 1 done (50.00%)
0:00 [1 rows, 34B] [10 rows/s, 342B/s]
```

## Describe `purchases` table

```
presto:default> describe purchases;
   Column   |  Type   | Comment 
------------+---------+---------
 accept     | varchar |         
 host       | varchar |         
 user-agent | varchar |         
 event_type | varchar |         
 timestamp  | varchar |         
(5 rows)

Query 20180404_224828_00010_zsma3, FINISHED, 1 node
Splits: 2 total, 1 done (50.00%)
0:00 [5 rows, 344B] [34 rows/s, 2.31KB/s]
```

## Query `purchases` table

```
presto:default> select * from purchases;
 accept |       host        |   user-agent    |   event_type   |        timestamp        
--------+-------------------+-----------------+----------------+-------------------------
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-04-04 22:36:13.124 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-04-04 22:36:13.128 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-04-04 22:36:13.131 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-04-04 22:36:13.135 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-04-04 22:36:13.138
 ...
 ```


# 
## Streaming

## Simpler spark

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    sword_purchases.printSchema()
    sword_purchases.show(100)


if __name__ == "__main__":
    main()
```

::: notes
- specify purchase_sword_event_schema schema in the job (before were inferring schema)
- rename is_purchase so specific to swords
- Combining a bunch of steps from before
- Not going to the rdd
- Start with raw events, filter whether they're sword purchases, 
- The from json has two pieces - the actual column and the schema
- select 3 columns: value, timestamp, the json created
- Best practices
- Robust against different schema, but that can be a gotcha if you end up with data that isn't formatted how you thought it was
- Streaming datasets don't allow you to access the rdd as we did before
- This isn't writing the hive table for us
:::


## Run

```
docker-compose exec spark spark-submit /w205/full-stack2/filter_swords_batch.py
```
::: notes

:::


## Turn that into a stream

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    query = sword_purchases \
        .writeStream \
        .format("console") \
        .start()

    query.awaitTermination()


if __name__ == "__main__":
    main()
```

::: notes
- In batch one, we tell it offsets, and tell it to read
- In streaming, no offsets
- query = and query.awaitTermination are differences
:::


## Run it

```
docker-compose exec spark spark-submit /w205/full-stack2/filter_swords_stream.py
```
::: notes
- run in stream,
- kick some events 
- feed it to automaically generate events, can see that grow in hdfs
:::

## Kick some more events

```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/purchase_a_sword
```

::: notes

```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
```
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
```
```
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```


## Write from a stream

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    sink = sword_purchases \
        .writeStream \
        .format("parquet") \
        .option("checkpointLocation", "/tmp/checkpoints_for_sword_purchases") \
        .option("path", "/tmp/sword_purchases") \
        .trigger(processingTime="10 seconds") \
        .start()

    sink.awaitTermination()


if __name__ == "__main__":
    main()
```

## Run it

```
docker-compose exec spark spark-submit /w205/full-stack2/write_swords_stream.py
```

## Feed it

```
while true; do
  docker-compose exec mids \
    ab -n 10 -H "Host: user1.comcast.com" \
      http://localhost:5000/purchase_a_sword
done
```

::: notes
```
while true; do docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword; done
```
:::

## Check what it wrote to Hadoop

```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```

# 
## down

    docker-compose down

::: notes
:::



#
## summary

## { data-background="images/pipeline-steel-thread-for-mobile-app.svg" } 




#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>
