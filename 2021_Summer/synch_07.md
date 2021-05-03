### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #7

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2021_Summer/checklist_b4_class_assignments.md

## Discuss Project 2: Tracking User Activity

Will probably defer this to the end of class to take into account what we do in class today.

## Spark Stack with Kafka

Setup
```
mkdir ~/w205/spark-with-kafka

cd ~/w205/spark-with-kafka

cp ~/w205/course-content/07-Sourcing-Data/docker-compose.yml .
```

Spin up the cluster
```
docker-compose up -d

docker-compose logs -f kafka
```

Create a topic 
```
docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should show
```
Created topic "foo".
```

Check the topic
```
docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper zookeeper:32181
```

Should show
```
Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1
```

Publish some stuff to Kafka
```
docker-compose exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list kafka:29092 --topic foo && echo 'Produced 42 messages.'"
```

Should show
```
Produced 42 messages.
```

Use kafkacat to read all the messages on the topic:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"
```

Run spark using the spark container
```
docker-compose exec spark pyspark
```

read stuff from kafka, at the pyspark prompt
```python
numbers = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","foo").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

See the schema
```python
numbers.printSchema()
```

Cast it as a strings
```python
numbers_as_strings=numbers.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

Take a look
```python
numbers_as_strings.show()

numbers_as_strings.printSchema()

numbers_as_strings.count()
```

To exit pyspark, we need to do a well formed scala call:
```
exit()
```

Use kafkacat to read all the messages on the topic:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"
```

down
```
docker-compose down
```

## Spark stack with Kafka with "real" messages

Pull data
```
cd ~/w205

curl -L -o github-example-large.json https://goo.gl/Y4MD58

cd ~/w205/spark-with-kafka
```

Spin up the cluster & check
```
docker-compose up -d

docker-compose logs -f kafka
```

Create a topic 
```
docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should see something like
```
Created topic "foo".
```

Check the topic
```
docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper zookeeper:32181
```

Should see something like
```
Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1
```

Check out our messages
```
docker-compose exec mids bash -c "cat /w205/github-example-large.json"

docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.'"
```

Individual messages
```
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c"
```

Publish some test messages to that topic with kafkacat
```
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"
```

Should see something like
```
Produced 100 messages.
```

Use kafkacat to read all the messages on the topic:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"
```

Run spark using the spark container
```
docker-compose exec spark pyspark
```

read stuff from kafka, at the pyspark prompt
```python
messages = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","foo").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

See the schema
```python
messages.printSchema()
```

See the messages
```python
messages.show()
```

Cast as strings
```python
messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

Take a look
```python
messages_as_strings.show()

messages_as_strings.printSchema()

messages_as_strings.count()
```

Unrolling json
```python
messages_as_strings.select('value').take(1)

messages_as_strings.select('value').take(1)[0].value

import json

first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)

first_message

print(first_message['commit']['committer']['name'])
```

To exit pyspark, we need to do a well formed scala call:
```
exit()
```

Use kafkacat to read all the messages on the topic:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"
```

Down
```
docker-compose down
```
