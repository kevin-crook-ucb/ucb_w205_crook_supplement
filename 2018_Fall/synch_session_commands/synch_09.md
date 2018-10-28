### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #9

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

Assignment-09 - Define your Pipeline

Assignment-10 - Setup Pipeline, Part 1

Assignment-11 - Setup Pipeline, Part 2

Assignment-12 - Synthesis Assignment

### Flask with Kafka

```
mkdir ~/w205/flask-with-kafka
cd ~/w205/flask-with-kafka
```

Setup
Create a docker-compose.yml with the following
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

  mids:
    image: midsw205/base:0.1.8
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

Spin up the cluster
```
docker-compose up -d
```

Create a topic events
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should show
```
Created topic "events".
```

Flask
Use the python flask library to write our simple API server
```python
#!/usr/bin/env python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def default_response():
    return "\nThis is the default response!\n"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    return "\nSword Purchased!\n"
    
```

Save this as 
```
~/w205/flask-with-fafka/game_api.py
```

run it via
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
```

Test it out
```
docker-compose exec mids curl http://localhost:5000/

docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

Stop flask
Kill flask with control-C

Generate events from our webapp
Let's add kafka into the mix 
```python
#!/usr/bin/env python
from kafka import KafkaProducer
from flask import Flask
app = Flask(__name__)
event_logger = KafkaProducer(bootstrap_servers='kafka:29092')
events_topic = 'events'

@app.route("/")
def default_response():
    event_logger.send(events_topic, 'default'.encode())
    return "\nThis is the default response!\n"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    # log event to kafka
    event_logger.send(events_topic, 'purchased_sword'.encode())
    return "\nSword Purchased!\n"
    
```
Run that
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
```

Test it out
Generate events
```
docker-compose exec mids curl http://localhost:5000/

docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

read from kafka
Use kafkacat to consume events from the events topic
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t events -o beginning -e"
```
down
```
docker-compose down
```
