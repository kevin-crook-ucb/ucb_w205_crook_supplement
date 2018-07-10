# Suggestions for Assignment 12 for MIDS W205 2018 Summer - Kevin Crook

Part of the Berkeley culture is to not place any limits on student success.  In keeping with this culture, there are open ended components of this assignment to allow students to synthesize what they learned in the assignment to take it to higher levels.  

I would like to keep it as open ended as possible, however, I understand students have skipped some of the minimum steps in the past, so I wanted to provide a checklist of minimum components. I'm also providing a list of suggested enhancements.

Also, remember that assignments 9, 10, 11, and 12 are part of the Project 3.  You will want to reuse as much content from the previous assignment.  

## Assignment 12 relates to Synchronous 12

## Minimum components

The following are the minimum components that I would be looking for in terms of a 9 (A).  These are the raw commands from class.  They may need adaptation to work for the differing json file and to be adapted for any enhancements you make:

```
mkdir ~/w205/full-stack/
cd ~/w205/full-stack
cp ~/w205/course-content/12-Querying-Data-II/docker-compose.yml .
cp ~/w205/course-content/12-Querying-Data-II/*.py .
```
```yml
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
    volumes:
      - ~/w205:/w205
    expose:
      - "8888"
    ports:
      - "8888:8888"
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
    extra_hosts:
      - "moby:127.0.0.1"
    command: bash

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
```
docker-compose up -d
```
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
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
```
docker-compose exec mids \
  env FLASK_APP=/w205/full-stack/game_api.py \
  flask run --host 0.0.0.0
```
```
docker-compose exec mids \
  kafkacat -C -b kafka:29092 -t events -o beginning
```
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/
    
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/purchase_a_sword

docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/

docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/purchase_a_sword
```
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
    event['Host'] = "moe"
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
```
docker-compose exec spark \
  spark-submit /w205/spark-from-files/separate_events.py
```
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


if __name__ == "__main__":
    main()
```
```
docker-compose exec spark \
  spark-submit /w205/full-stack/just_filtering.py
```
```python
@app.route("/purchase_a_knife")
def purchase_a_knife():
    purchase_knife_event = {'event_type': 'purchase_knife',
                            'description': 'very sharp knife'}
    log_to_kafka('events', purchase_knife_event)
    return "Knife Purchased!\n"
```
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
```
docker-compose exec spark \
  spark-submit /w205/full-stack/filtered_writes.py
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/
docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```
```
docker-compose exec spark \
  env \
    PYSPARK_DRIVER_PYTHON=jupyter \
    PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' \
  pyspark
```
```
purchases = spark.read.parquet('/tmp/purchases')
purchases.show()
purchases.registerTempTable('purchases')
purchases_by_example2 = spark.sql("select * from purchases where Host = 'user1.comcast.com'")
purchases_by_example2.show()
df = purchases_by_example2.toPandas()
df.describe()
```
```
docker-compose down
```

## Suggestions for annotating the minimum components:

* Everything professionally formatted in markdown.

* Annotate the architecture as defined in the yml file.  Describe each container and what it does.

* For each step, include a header, sentences to describe the step, and the step.  If the step is long, you may want to show it multi-line as I do in mine.

* Steps can occur at several levels: droplet command line, container command line, interactive python, python files, pyspark, jupyter notebook, etc.  Be sure you include all of the steps.  For example, don't just say we ran pyspark.  Give the details of what we did in pyspark and annotate. 

* For json files, pull out an individual json object and show it as an example of what the file looks like.

* For csv files, pull out the header and a couple of lines to show it as an example of what the file looks like.

## Suggestions for enhancements

**Be sure to include an Enchancement section at the end of your submission, preferably with a bullet list of enhancements where each enchance is a bullet point with a brief description.**

Assignments 9, 10, 11, and 12 are part of Project 3.  You can reuse enhancements in the next assignemnt.

Enhancements are totally open ended, so feel free to add any enhancements you wish, as I do not want to place any limits on your success.  You would need to do substantial enhancements above the minimum in terms of a 10 (A+):

* Executive summary

* Introduction

* Architecure (some included architecture diagrams they created externally and uploaded images to include in mark down)

* Add steps to explore and understand further in additional to the minimum steps.

* We are doing the data engineering piece and handing it over to the data scientists.  What types of analytics could the data scientists do? 

* Other business problems this same technology can be used for.

* Furture enhancements that are possible but we didn't have time to build out.

* Summary of findings, wrap up.

* Appendix: List of Enhancements (be sure to include this at the end for grading purposes)
