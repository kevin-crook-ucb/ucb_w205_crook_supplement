# Work in progress - please wait for it to be finalized


# Kevin Crook's week 11 synchronous session supplemental notes

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
* Some misc loose ends
  * Discuss having a Data Science Portfolio of your work
    * GitHub public repo
    * README.md should be informative with links to your resume in pdf, outline of areas and example
    * Consider directories such as Resume, Machine Learning, Deep Learning, Natural Language Processing, Data Visualization, Blockchain Analytics
    * Suggestion: take assignment 5, the Jupyter Notebook which queries Google Big Query and do some analytics on the Bitcoin dataset and make it look professional using markdown cells, pandas tables, data visualizations, etc.
  * Using Jupyter Notebook with Spark from Chrome on your laptop to connect to your droplet => docker cluster => spark container
  * Using a web browser from your laptop to connect to our flask web server and issue API commands
  * Using telnet from the droplet to connect to our flask web server and issue API commands
  * Using telnet from your laptop to connect to our flask web server and issue API commands
  * Using PuTTY from your laptop to connect to our flask web server and issue API commands
* Activity 
  * Purpose: So far in past weeks, we have 
* Time permitting - remainder of class time for group meetings

---
title: Fundamentals of Data Engineering
author: Week 11 - sync session
...

---

 

#
## Assignment Review
- Review your Assignment 10
- Get ready to share
- `docker pull midsw205/base:latest`
- `git pull` in `~/w205/course-content`

::: notes

#### Breakout at about 5 after the hour:
- Check in with each group 
- have students share screen
:::


## Due Friday (PR)

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
## Project 3 Group Breakout

- Plan for project
- Which events will you include?
- Which parameters will you include?
- What will you track the state of?
- How will you need to change:
  * flask app code?
  * pyspark code?
  * code to implement tracking state?


::: notes
- If a person isn't doing in group, put together groups of people doing that option, to brainstorm/plan for class.
- Which option:
- All: Game shopping cart data used for homework (flask app) 
- Advanced option 1: Generate (in flask) and filter (in spark) more types of items.
- Advanced option 2: Enhance the API (in flask) to accept parameters for purchases (sword/item type) and filter (in spark) 
- Advanced option 3: Shopping cart data & track state (e.g., user's inventory) and filter (in spark) 

:::


# 
## Running Spark Jobs


#
## Setup

## Set up directory, get docker-compose
```
mkdir ~/w205/spark-from-files/
cd ~/w205/spark-from-files
cp ~/w205/course-content/11-Storing-Data-III/docker-compose.yml .
cp ~/w205/course-content/11-Storing-Data-III/*.py .
```

## The `docker-compose.yml` 

Create a `docker-compose.yml` with the following
```yaml
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
    image: midsw205/cdh-minimal:latest
    expose:
      - "8020" # nn
      - "50070" # nn http
      - "8888" # hue
    #ports:
    #- "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"

  spark:
    image: midsw205/spark-python:0.0.5
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
    volumes:
      - "~/w205:/w205"
    command: bash
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "5000"
    ports:
      - "5000:5000"
    volumes:
      - "~/w205:/w205"
    extra_hosts:
      - "moby:127.0.0.1"
```

::: notes
- no need for a datafile on this one.
- Walk through the docker-compose.yml file
:::


## Spin up the cluster

```
docker-compose up -d
```

::: notes
Now spin up the cluster
```
docker-compose up -d
```
:::

## Wait for things to come up
```
docker-compose logs -f cloudera
```
::: notes
- Go through what's happening in logs
```
docker-compose logs -f cloudera
```
:::

## Check out hadoop

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

::: notes
Let's check out hdfs before we write anything to it
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
:::

## Create a topic

```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic events \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists --zookeeper zookeeper:32181
```

::: notes
- First, create a topic `events`
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
:::

## Should show

    Created topic "events".


#
## Flask

## Take our flask app - with request.headers

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

## Run it
```
docker-compose exec mids \
  env FLASK_APP=/w205/spark-from-files/game_api.py \
  flask run --host 0.0.0.0
```

::: notes
```
docker-compose exec mids env FLASK_APP=/w205/spark-from-files/game_api.py flask run --host 0.0.0.0
```
:::

## Generate events from browser
- localhost:5000/
- localhost:5000/purchase_a_sword

::: notes
```
    docker-compose exec mids curl http://localhost:5000/
    docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```
:::

## Read from kafka
```
docker-compose exec mids \
  kafkacat -C -b kafka:29092 -t events -o beginning -e
```

::: notes
ok to skip this

```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```
:::

## Should see 

```
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
...
```


#
## Spark

## Capture our pyspark code in a file this time

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession


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

    events = raw_events.select(raw_events.value.cast('string'))
    extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()

    extracted_events \
        .write \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

::: notes
- Same as before, but you need to create a spark session when you use spark submit
- What's a spark session?
- add `printSchema()` and `show()` liberally throughout the rest of these examples
- [optional] run against an empty topic first to show spark exceptions
:::

## run it

```
docker-compose exec spark \
  spark-submit \
    /w205/spark-from-files/extract_events.py
```

::: notes
```
docker-compose exec spark spark-submit /w205/spark-from-files/extract_events.py
```
:::

## if you didn't generate any events

```
Traceback (most recent call last):
  File "/w205/spark-from-files/extract_events.py", line 35, in <module>
    main()
  File "/w205/spark-from-files/extract_events.py", line 27, in main
    extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()
  File "/spark-2.2.0-bin-hadoop2.6/python/lib/pyspark.zip/pyspark/sql/session.py", line 57, in toDF
  File "/spark-2.2.0-bin-hadoop2.6/python/lib/pyspark.zip/pyspark/sql/session.py", line 535, in createDataFrame
  File "/spark-2.2.0-bin-hadoop2.6/python/lib/pyspark.zip/pyspark/sql/session.py", line 375, in _createFromRDD
  File "/spark-2.2.0-bin-hadoop2.6/python/lib/pyspark.zip/pyspark/sql/session.py", line 346, in _inferSchema
  File "/spark-2.2.0-bin-hadoop2.6/python/lib/pyspark.zip/pyspark/rdd.py", line 1364, in first
ValueError: RDD is empty
```

## check out results in hadoop

```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and
```
    docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
```

::: notes
:::


#
## Deploying a Spark job to a cluster

##

```
docker-compose exec spark spark-submit filename.py
```

is really just


```
docker-compose exec spark \
  spark-submit \
    --master 'local[*]' \
    filename.py
```

::: notes
To submit a spark job to a cluster, you need a "master"
:::

## standalone

```
docker-compose exec spark \
  spark-submit \
    --master spark://23.195.26.187:7077 \
    filename.py
```
(this won't work here)

::: notes
In a spark standalone cluster, there are multiple containers... a single
spark "master" and many spark "workers"

To submit a job, you point to the spark "master"

Of course, in `docker-compose` you'd use a dns name `spark://spark-master:7077`
:::

## yarn

```
docker-compose exec spark \
  spark-submit \
    --master yarn \
    --deploy-mode cluster \
    filename.py
```
(this won't work here)

::: notes
In a yarn cluster, you tell it to talk to the cluster's ResourceManager
(address is usually already set in hadoop config)
:::

## mesos

```
docker-compose exec spark \
  spark-submit \
    --master mesos://mesos-master:7077 \
    --deploy-mode cluster \
    filename.py
```
(this won't work here)

::: notes
In a mesos cluster, you tell it to talk to the mesos master
:::

## kubernetes

```
docker-compose exec spark \
  spark-submit \
    --master k8s://kubernetes-master:443 \
    --deploy-mode cluster \
    filename.py
```
(this won't work here)


#
## More Spark!

##

```python
#!/usr/bin/env python
"""Extract events from kafka, transform, and write to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe" # silly change to show it works
    event['Cache-Control'] = "no-cache"
    return json.dumps(event)


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

    munged_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .withColumn('munged', munge_event('raw'))
    munged_events.show()

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()
    extracted_events.show()

    extracted_events \
        .write \
        .mode("overwrite") \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

::: notes
Here's an example that allows arbitrary tranformation of the json `value`
_before_ extraction.
:::


## Let's look at separating events

##

```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe" # silly change to show it works
    event['Cache-Control'] = "no-cache"
    return json.dumps(event)


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

    munged_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .withColumn('munged', munge_event('raw'))

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()

    sword_purchases = extracted_events \
        .filter(extracted_events.event_type == 'purchase_sword')
    sword_purchases.show()
    # sword_purchases \
        # .write \
        # .mode("overwrite") \
        # .parquet("/tmp/sword_purchases")

    default_hits = extracted_events \
        .filter(extracted_events.event_type == 'default')
    default_hits.show()
    # default_hits \
        # .write \
        # .mode("overwrite") \
        # .parquet("/tmp/default_hits")


if __name__ == "__main__":
    main()
```

::: notes
And here's one that filters out multiple events.

This works, but...

Question:  What happens with events that have different schema?

- Problem here is if you're exploding flat json, you'll have some decisions to make

:::


#
## Remember to tear down your cluster

    docker-compose down


#

## Summary

## { data-background="images/pipeline-steel-thread-for-mobile-app.svg" } 

::: notes
(repeat from earlier)

Let's walk through this
- user interacts with mobile app
- mobile app makes API calls to web services
- API server handles requests:
    - handles actual business requirements (e.g., process purchase)
    - logs events to kafka
- spark then:
    - pulls events from kafka
    - filters/flattens/transforms events
    - separates event types
    - writes to storage
- presto then queries those events
:::


#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>

  
