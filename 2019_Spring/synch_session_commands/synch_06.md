### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #6

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md


#### Pipes
```
cat junk.csv | sort | uniq | wc -l
```

#### Discuss Project 2: Tracking User Activity

Assignment 6 - Get and Clean Data

Assignment 7 - Setup Pipeline

Assignment 8 - Build and Write-up Pipeline

#### Kafka

Create a kafka directory and change to it:
```
mkdir ~/w205/kafka
cd ~/w205/kafka
```

vi the docker-compose.yml and insert the following
```yml
---
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    network_mode: host
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    extra_hosts:
      - "moby:127.0.0.1"

  kafka:
    image: confluentinc/cp-kafka:latest
    network_mode: host
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: localhost:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    extra_hosts:
      - "moby:127.0.0.1"
```

Docker compose spin things up
```
cd ~/w205/kafka
docker-compose up -d
docker-compose ps
```

Should see something like
```
    Name                        Command            State   Ports
    -----------------------------------------------------------------------
    kafkasinglenode_kafka_1       /etc/confluent/docker/run   Up
    kafkasinglenode_zookeeper_1   /etc/confluent/docker/run   Up
```

Check zookeeper
```
docker-compose logs zookeeper | grep -i binding
```

Should see something like:
```
zookeeper_1  | [2016-07-25 03:26:04,018] INFO binding to port 0.0.0.0/0.0.0.0:32181 
(org.apache.zookeeper.server.NIOServerCnxnFactory)
```

Check the kafka broker
```
docker-compose logs kafka | grep -i started
```

Should see something like
```

    kafka_1      | [2017-08-31 00:31:40,244] INFO [Socket Server on Broker 1], Started 1 acceptor threads (kafka.network.SocketServer)
    kafka_1      | [2017-08-31 00:31:40,426] INFO [Replica state machine on controller 1]: Started replica state machine with initial state -> Map() (kafka.controller.ReplicaStateMachine)
    kafka_1      | [2017-08-31 00:31:40,436] INFO [Partition state machine on Controller 1]: Started partition state machine with initial state -> Map() (kafka.controller.PartitionStateMachine)
    kafka_1      | [2017-08-31 00:31:40,540] INFO [Kafka Server 1], started (kafka.server.KafkaServer)
```

Create a Topic foo
```
docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181
```

Should see something like
```
Created topic "foo".
```

Check the topic
```
docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper localhost:32181
```

Should see something like
```
    Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1
```

Publish messages
```
docker-compose exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list localhost:29092 --topic foo && echo 'Produced 42 messages.'"
```

Should see something like
```
Produced 42 messages.
```

Consume Messages
```
docker-compose exec kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic foo --from-beginning --max-messages 42
```

Should see something like
```
    1
    ....
    42
    Processed a total of 42 messages
```

Tear things down
```
docker-compose down
```

#### Kafka with "real" messages / Kafka with json example

Copy the docker-compose.yml from the course-content repo
```
cp ~/w205/course-content/06-Transforming-Data/docker-compose.yml ~/w205/kafka/
```

Pull data
```
cd ~/w205/kafka/
curl -L -o github-example-large.json https://goo.gl/Y4MD58
```

Spin up the cluster
``` 
docker-compose up -d
```

Watch it come up
```
docker-compose logs -f kafka
```

create a topic
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
docker-compose exec mids bash -c "cat /w205/kafka/github-example-large.json"
docker-compose exec mids bash -c "cat /w205/kafka/github-example-large.json | jq '.'"
docker-compose exec mids bash -c "cat /w205/kafka/github-example-large.json | jq '.[]' -c"
```

Publish some test messages to that topic with the kafka console producer
```
docker-compose exec mids bash -c "cat /w205/kafka/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"
```

Should see something like
```
Produced 100 messages.
```

Consume the messsages

We can either do what we did before
```
docker-compose exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic foo --from-beginning --max-messages 42
```

or
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"
```

and maybe
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e" | wc -l
```

Down
```
docker-compose down
```

