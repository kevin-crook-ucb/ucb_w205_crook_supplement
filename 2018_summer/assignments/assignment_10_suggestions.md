# Suggestions for Assignment 10 for MIDS W205 2018 Summer - Kevin Crook

Part of the Berkeley culture is to not place any limits on student success.  In keeping with this culture, there are open ended components of this assignment to allow students to synthesize what they learned in the assignment to take it to higher levels.  

I would like to keep it as open ended as possible, however, I understand students have skipped some of the minimum steps in the past, so I wanted to provide a checklist of minimum components. I'm also providing a list of suggested enhancements.

Also, remember that assignments 9, 10, 11, and 12 are part of the Project 3.  You will want to reuse as much content from the previous assignment.  

## Assignment 10 relates to Synchronous 10

## Minimum components

The following are the minimum components that I would be looking for in terms of a 9 (A).  These are the raw commands from class.  They may need adaptation to work for the differing json file and to be adapted for any enhancements you make:

```
mkdir ~/w205/flask-with-kafka-and-spark/
```
```
cd ~/w205/flask-with-kafka-and-spark/
```
```
cp ~/w205/course-content/10-Transforming-Streaming-Data/docker-compose.yml .
```
```
docker-compose up -d
```
```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic events \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists --zookeeper zookeeper:32181
```
```python
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "\nThis is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "\nSword Purchased!\n"
    
```
```
docker-compose exec mids \
  env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_json_events.py \
  flask run --host 0.0.0.0
```
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids \
  kafkacat -C -b kafka:29092 -t events -o beginning -e
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
    return "\nThis is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "\nSword Purchased!\n"
    
```
```
docker-compose exec mids \
  env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_extended_json_events.py \
  flask run --host 0.0.0.0
```
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```
```
docker-compose exec mids \
  kafkacat -C -b kafka:29092 -t events -o beginning -e
```
```
docker-compose exec spark pyspark
```
```python
raw_events = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","events") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```
```python
raw_events.cache()
```
```python
events = raw_events.select(raw_events.value.cast('string'))
```
```python
import json
extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()
```
```python
extracted_events.show()
```    
```python
exit()
```
```
control-C
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
