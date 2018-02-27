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

If you want to see the kafka logs live as it comes up use the following command. The -f tells it to keep checking the file for any new additions to the file and print them.  To stop this command, use a control-C: 
```
docker-compose logs -f kafka
```

Create a topic called foo in the kafka container using the kafka-topisc utility:
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

The same command on 1 line to make it easy to copy and paste:
```
docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

You should see the following message to let us know the topic foo was created correctly
```
Created topic "foo".
```

Check the topic in the kafka container using the kafka-topics utility:
```
docker-compose exec kafka \
  kafka-topics \
  --describe \
  --topic foo \
  --zookeeper zookeeper:32181
```

The same command on 1 line to make it easy to copy and paste:
```
docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper zookeeper:32181
```

You should see the following or similar output from the previous command:
```
Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1
```

Now that we have the topic created in kafka, we want to publish the numbers from 1 to 42 to that topic.  We use the kafka container with the kafka-console-producer utility:
```
docker-compose exec kafka \
  bash -c "seq 42 | kafka-console-producer \
    --request-required-acks 1 \
    --broker-list kafka:29092 \
    --topic foo && echo 'Produced 42 messages.'"
```

The same command on 1 line to make it easy to copy and paste:
```
docker-compose exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list kafka:29092 --topic foo && echo 'Produced 42 messages.'"
```

You should see the following or similar output from the previous command:
```
Produced 42 messages.
```

In the spark container, run the python spark command line utility called pyspark:
```
docker-compose exec spark pyspark
```

Using pyspark, we will write some python spark code to consumer from the kafka topic:
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

The same command on 1 line to make it easy to copy and paste:
```
numbers = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","foo").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

Print the schema for the RDD:
```
numbers.printSchema()
```

Create a new RDD which stores the numbers as strings.  Note the RDD's are immutable, so we cannot change them in place, we have to make a copy:
```
numbers_as_strings=numbers.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

Use some of the methods of the RDD to display various things:

Display the entire RDD:
```
numbers_as_strings.show()
```

Display the schema for the RDD:
```
numbers_as_strings.printSchema()
```

Display the number of items in the RDD:
```
numbers_as_strings.count()
```

Exit pyspark using:
```
exit()
```

Tear down the docker cluster:
```
docker-compose down
```

Verify the docker cluster is down:
```
docker-compose ps -a
```

## Activity 2

Continue in our same directory with the same yml file:
```
~/w205/spark-with-kafka
```

Download our GitHub example data from the Internet using the curl utility.  Note that since it's using HTTPS, you can paste the URL into a web browser to test if the download works or not.  This is always highly recommended.  It should produce a json file.  However, if there are any errors, it will produce an XML file:
```
curl -L -o github-example-large.json https://goo.gl/Hr6erG
```

Start the docker cluster:
```
docker-compose up -d
```

(as far as I've gotten with cleaning up the formatting - Kevin)


As before 
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

## Down

    docker-compose down


