## UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #13

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Summer/synch_session_commands/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

Assignment-09 - Define your Pipeline

Assignment-10 - Setup Pipeline, Part 1

Assignment-11 - Setup Pipeline, Part 2

Assignment-12 - Synthesis Assignment

### Flask-Fafka-Spark-Hadoop-Presto Part II

Get Started
```
mkdir ~/w205/full-stack2/

cd ~/w205/full-stack2

cp ~/w205/course-content/13-Understanding-Data/docker-compose.yml .

cp ~/w205/course-content/13-Understanding-Data/*.py .
```

Flask-Fafka-Spark-Hadoop-Presto Part II

Setup

Spin up the cluster
```
docker-compose up -d
```

Web-app
* Take our instrumented web-app from before ```~/w205/full-stack/fame_api.py```

run flask
```
docker-compose exec mids env FLASK_APP=/w205/full-stack2/game_api.py flask run --host 0.0.0.0
```

Set up to watch kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```

Apache Bench to generate data
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```

Some Spark to Write Events

Run this
```
docker-compose exec spark spark-submit /w205/full-stack2/filtered_writes.py
```

See purchases in hdfs
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```

Queries from Presto

Hive metastore
* Track schema
* Create a table

Hard Way
```
docker-compose exec cloudera hive
```

```
create external table if not exists default.purchases2 (Accept string, Host string, User_Agent string, event_type string, timestamp string) stored as parquet location '/tmp/purchases'  tblproperties ("parquet.compress"="SNAPPY");
```

Or... we can do this an easier way
```
docker-compose exec spark pyspark
```

```python
df = spark.read.parquet('/tmp/purchases')

df.registerTempTable('purchases')

query = "create external table purchase_events stored as parquet location '/tmp/purchase_events' as select * from purchases"

spark.sql(query)
```

Can just include in job

Run this
```
docker-compose exec spark spark-submit /w205/full-stack2/write_hive_table.py
```

See it wrote to hdfs
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```

and now ...
* Query this with presto
```
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```

What tables do we have in Presto?
```
show tables;
```

Describe purchases table
```
describe purchases;
```

Query purchases table
```
select * from purchases;
```

Streaming

Simpler spark

Run

```
docker-compose exec spark spark-submit /w205/full-stack2/filter_swords_batch.py
```

Turn that into a stream

Run it
```
docker-compose exec spark spark-submit /w205/full-stack2/filter_swords_stream.py
```

Kick some more events
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```

Write from a stream

Run it
```
docker-compose exec spark spark-submit /w205/full-stack2/write_swords_stream.py
```

Feed it
```
while true; do docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword; done
```

Check what it wrote to Hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```

down
```
docker-compose down
```
