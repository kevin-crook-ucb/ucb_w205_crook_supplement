# Kevin Crook's week 9 synchronous session supplemental notes

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
    * docker pull midsw205/base:latest
* Activity
  * Purpose: setup a web server running a simple web API service which will service web API calls by writing them to a kafka topic, using curl make web API calls to our web service to test, manually consume the kafka topic to verify our web service is working.  subject matter is a mobile game developer who sells game events such as purcase a sword, purchase a knife, join a guild, etc.  In our mids container we will use flask, which is a simple lightweight python based web server.
  * Create a docker cluster with 3 containers: zookeeper, kafka, and mids
  * Create a kafka topic called events
  * Install flask into out mids container
  * Write a python scipt using the flash module to implement a simple web service and print the results to standard output
  * Run our python script in the mids container
  * Using a curl, we will make some web API calls manually
  * Stop flask
  * Beef up our python script using flash to write to the kafka topic instead of standard output.  We will use the KafkaProducer class in python.
  * Run our beefed up python script in the mids container
  * Using curl, we will make some web API calls manually
  * Consume the kafka topic events
  * stop flask and tear down the cluster
* Project 3 overview
* Assignment 9 overview
    
# Activity

Create a directory:
```
mkdir ~/w205/flask-with-kafka
cd ~/w205/flask-with-kafka
```

Create a `docker-compose.yml` with the following.  Remember to fix the drive mappings if needed:
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

Start the docker cluster:
```
docker-compose up -d
```

Create the kafka topic events (as we have done before):
```
 docker-compose exec kafka \
   kafka-topics \
     --create \
     --topic events \
     --partitions 1 \
     --replication-factor 1 \
     --if-not-exists \
     --zookeeper zookeeper:32181
```

The same command on 1 line for convenience:
```
 docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should see the following output:
```
Created topic "events".
```

Example scenario:
* You're a mobile game developer.  During gameplay, your users perform various
actions such as
  * purchase a sword
  * purchase a knife
  * join a guild
* To process these actions, your mobile app makes API calls to a web-based
API-server.  

We will use the python flask module to write a simple API server.  

- Use the python `flask` library to write our simple API server. Create a file `~/w205/flask-with-kafka/game_api.py` with the following python code:
```python
#!/usr/bin/env python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def default_response():
    return "This is the default response!"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    return "Sword Purchased!"
```

Run the python script using the following command.  This will tie up this linux command line window.  We will see output from our python program here as we make our web API calls:
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
```

Using another linux command line window, use curl to make web API calls. Note that TCP port 5000 is the port we are using.
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

In the flask window, stop flask with a control-C

Edit our python flask script to publish to the kafka topic in a addition to writing to standard output:
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
    return "This is the default response!"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    # log event to kafka
    event_logger.send(events_topic, 'purchased_sword'.encode())
    return "Sword Purchased!"
```

Run the python flask script as before:
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
```

In another linux command line windows, use curl to make web API calls.
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

Use kafkacat to consume the messages that our web service wrote to the kafka topic:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t events -o beginning -e"
```

Optional, if you want, go back and generate more web API calls and consume the topic to see how they show up.

In the flask window, stop flask with a control-C

Tear down the docker cluster:
```
docker-compose down
```
