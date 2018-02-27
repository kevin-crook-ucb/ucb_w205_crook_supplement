# Kevin Crook's week 7 synchronous session supplemental notes

Overview of today's synch session:
* Everyone attempt to login and use their Digital Ocean droplet.  Separate instructions are provided so they can be referred to in the future.  Some students may not be able to use them due to blocked ports or admin rights to run PuTTY.
* Review assignments 6, 7, and 8
* Activity 1:
  * Purpose: last week, we created a kafka topic, published numbers to the topic, and then consumed the numbers from the topic using core python.  In this activity, we are doing the same thing, except instead of reading using core python, we will be using spark via the python interface called pyspark.
  * Create a docker cluster with 4 containers: zookeeper, kafka, mids, and spark.
  * Create a kafka topic and publish the numbers 1 to 42 to that topic
  * Start pyspark (python spark interface in the spark container) and write spark code in python to consume the messages from the topic
  * tear down the cluster
* Activity 2:
  * Purpose: last week, we created a kafka topic, published json data to the topic, and then consumed the json data using core python with pandas.  In this activity, we are doing the same thing, except instead of using core python with pandas, we will be using spark via the python interface called pyspark.
  * Same docker cluster as above.
  * Pull down a json format file of GitHub data
  * Create a kafka topic and publish the GitHub data to the topic
  * Start pyspark and write spark code in python to consume the messages from the topic and place them into a spark RDD (Resilient Distributed Dataset).  An RDD is essentially a parallel list.
  * Do some manipulations to filter and format the data in the spark RDD.
  * tear down the cluster
  
## Activity 1

Create a directory for spark with kafka and change to that directory:
```
mkdir ~/w205/spark-with-kafka
cd ~/w205/spark-with-kafka
```

Copy the yml file to the spark with kafka directory we should be in:
```
cp ~/w205/course-content/07-Sourcing-Data/docker-compose.yml .
```

Edit the yml file we just copied.  During class we will talk through it.  For volume mounts, you may need to change them to fully qualified path names.  If you are running Windows desktop, you will need to change them to a fully qualified Windows path:

Start the docker cluster:

```
docker-compose up -d
```

If you want to see the kafka logs live as it comes up use the following command: 

```
docker-compose logs -f kafka
```

::: notes
Now spin up the cluster
```
docker-compose up -d
```
and watch it come up
```
    docker-compose logs -f kafka
```
when this looks like it's done, you can safely detach with `Ctrl-C`.

:::


## create a topic

```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic foo \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists \
    --zookeeper zookeeper:32181
```

::: notes
First, create a topic `foo`

docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
:::

## Should show

    Created topic "foo".

## Check the topic

```
docker-compose exec kafka \
  kafka-topics \
  --describe \
  --topic foo \
  --zookeeper zookeeper:32181
```

## Should show

    Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1



## Publish some stuff to kafka

```
docker-compose exec kafka \
  bash -c "seq 42 | kafka-console-producer \
    --request-required-acks 1 \
    --broker-list kafka:29092 \
    --topic foo && echo 'Produced 42 messages.'"
```

::: notes
Use the kafka console producer to publish some test messages to that topic

docker-compose exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list kafka:29092 --topic foo && echo 'Produced 42 messages.'"

:::

## Should show

    Produced 42 messages.


#
## Run spark using the `spark` container

```
docker-compose exec spark pyspark
```

::: notes
Spin up a pyspark process using the `spark` container

docker-compose exec spark pyspark

We have to add some kafka library dependencies on the cli for now.
:::

## read stuff from kafka

At the pyspark prompt,

```
numbers = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","foo") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```

::: notes
At the pyspark prompt,

read from kafka

numbers = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","foo") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 

:::

## See the schema

```
numbers.printSchema()
```

## Cast it as strings 

```
numbers_as_strings=numbers.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

::: notes
cast it as strings (you can totally use `INT`s if you'd like)

numbers_as_strings=numbers.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
:::

## Take a look

```
numbers_as_strings.show()
```
```
numbers_as_strings.printSchema()
```

```
numbers_as_strings.count()
```
::: notes
then you can exit pyspark using either `ctrl-d` or `exit()`.

`numbers_as_strings.show()`

numbers_as_strings.printSchema()

:::


## down
```
docker-compose down
```


#
## Spark stack with Kafka with "real" messages


::: notes
:::

## docker-compose.yml file

- same 
- still in your `~/w205/spark-with-kafka`

::: notes
- same `docker-compose.yml` from above
:::


## Pull data

```
curl -L -o github-example-large.json https://goo.gl/Hr6erG
```

## Spin up the cluster & check

```
docker-compose up -d
```

```
docker-compose logs -f kafka
```

::: notes
when this looks like it's done, detach with `Ctrl-C`
:::


## create a topic

```
docker-compose exec kafka \
  kafka-topics \
    --create \
	  --topic foo \
	  --partitions 1 \
	  --replication-factor 1 \
	  --if-not-exists \
	  --zookeeper zookeeper:32181
```

::: notes
docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
:::

## Should see something like

    Created topic "foo".

## Check the topic


```
docker-compose exec kafka \
  kafka-topics \
    --describe \
    --topic foo \
    --zookeeper zookeeper:32181
```

::: notes
docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper zookeeper:32181
:::
## Should see something like

    Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1



#
## Publish some stuff to kafka

## Check out our messages

```
docker-compose exec mids bash -c "cat /w205/github-example-large.json"
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.'"
```

::: notes
ugly 1st
pretty print
:::


## Individual messages

```
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c"
```

::: notes
Go over | jq stuff
:::

## Publish some test messages to that topic with the kafka console producer

```
docker-compose exec mids \
  bash -c "cat /w205/github-example-large.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"
```

::: notes
docker-compose exec mids bash -c "cat /w205/github-example-large.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"
:::

## Should see something like

    Produced 100 messages.





#
## Run spark using the `spark` container

```
docker-compose exec spark pyspark
```

::: notes
Spin up a pyspark process using the `spark` container

docker-compose exec spark pyspark

We have to add some kafka library dependencies on the cli for now.
:::

## read stuff from kafka

At the pyspark prompt,

```
messages = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","foo") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```

::: notes
At the pyspark prompt,

read from kafka

messages = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","foo") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 

:::

## See the schema

```
messages.printSchema()
```

## See the messages

```
messages.show()
```

## Cast as strings 


```
messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

::: notes
cast it as strings (you can totally use `INT`s if you'd like)

messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
:::

## Take a look

```
messages_as_strings.show()
```

```
messages_as_strings.printSchema()
```

```
messages_as_strings.count()
```

::: notes
then you can exit pyspark using either `ctrl-d` or `exit()`.

`messages_as_strings.show()`

messages_as_strings.printSchema()

messages_as_strings.count()
:::

## Unrolling json
```
messages_as_strings.select('value').take(1)
```

```
messages_as_strings.select('value').take(1)[0].value
```

```
import json
```

```
first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)
```

```
first_message
```

```
print(first_message['commit']['committer']['name'])
```

::: notes
messages_as_strings.select('value').take(1)

messages_as_strings.select('value').take(1)[0].value
>>> import json
>>> first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)
>>> first_message
>>> print(first_message['commit']['committer']['name'])
Nico Williams
:::

## Breakout

- Change around some of the fields to print different aspects of the commit

## Down

    docker-compose down


#
## Assignment 07
- Step through this process using the Project 2 data
- What you turn in:
- In your `/assignment-07-<user-name>` repo:
  * your `docker-compose.yml` 
  * once you've run the example on your terminal
    * Run `history > <user-name>-history.txt`
    * Save the relevant portion of your history as `<user-name>-annotations.md`
    * Annotate the file with explanations of what you were doing at each point.



#
## Summary

![](images/streaming-bare.svg){style="border:0;box-shadow:none"}


::: notes
- Review what we just did
:::



#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>

