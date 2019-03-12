## UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #11

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Spring/synch_session_commands/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

Assignment-09 - Define your Pipeline

Assignment-10 - Setup Pipeline, Part 1

Assignment-11 - Setup Pipeline, Part 2

Assignment-12 - Synthesis Assignment

### Running Spark Jobs

Setup
Setup directory, get docker-compose
```
mkdir ~/w205/spark-from-files/

cd ~/w205/spark-from-files

cp ~/w205/course-content/11-Storing-Data-III/docker-compose.yml .

cp ~/w205/course-content/11-Storing-Data-III/*.py .
```

Spin up the cluster
```
docker-compose up -d
```

Wait for things to come up
```
docker-compose logs -f cloudera
```

Check out hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

Create a topic
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should show
```
Created topic "events".
```

Flask
Take our flask app - with request.headers
Run it
```
docker-compose exec mids env FLASK_APP=/w205/spark-from-files/game_api.py flask run --host 0.0.0.0
```

Generate events from browser (where xxxx is your IP address)
```
http://xxxxx:5000/

http://xxxxx:5000/purchase_a_sword
```

Read from kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```

Should see
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

Spark

Capture our pyspark code in a file tyis time

run it
```
docker-compose exec spark spark-submit /w205/spark-from-files/extract_events.py
```

if you didn't generate any events
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

check out results in hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
```

Deploying a Spark job to a cluster 
standalone (this won't work here)
yarn (this won't work here)
mesos (this won't work here)
kubernetes (this won't work here)

More Spark!

```
docker-compose exec spark spark-submit /w205/spark-from-files/transform_events.py
```

Let's look at separating events
```
docker-compose exec spark spark-submit /w205/spark-from-files/separate_events.py
```

Tear down the cluster:
```
docker-compose down
```
