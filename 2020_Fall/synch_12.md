## UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #12

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2020_Fall/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

We will wait and discuss project 3 at the end of class.  It covers weeks 11, 12, and 13.  


### Flask-Kafka-Spark-Hadoop-Presto Part I

Setup
Set up directory, get docker-compose
```
mkdir ~/w205/full-stack/

cd ~/w205/full-stack

cp ~/w205/course-content/12-Querying-Data-II/docker-compose.yml .

cp ~/w205/course-content/12-Querying-Data-II/*.py .
```

Spin up the cluster
```
docker-compose up -d
```

Create a topic events
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Web-app
Take our instrumented web-app from before
`~/w205/full-stack/game_api.py`

run flask
```
docker-compose exec mids env FLASK_APP=/w205/full-stack/game_api.py flask run --host 0.0.0.0
```

Setup to watch kafka
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

More Spark

what if different event types have different schema?
`~/w205/full-stack/just_filtering.py`

run this
```
docker-compose exec spark spark-submit /w205/full-stack/just_filtering.py
```

we can play with this
add a new event type to the flask app...
```python
@app.route("/purchase_a_knife")
def purchase_a_knife():
    purchase_knife_event = {'event_type': 'purchase_knife',
                            'description': 'very sharp knife'}
    log_to_kafka('events', purchase_knife_event)
    return "Knife Purchased!\n"
```

Add some purchase knife events, which will have a different schema:
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_knife

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_knife
```

run this
```
docker-compose exec spark spark-submit /w205/full-stack/filtered_writes.py
```

should see purchases in hdfs
```
docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```

Queries from Spark

create a symbolic link so we can access our mounted w205 directory:
```
docker-compose exec spark bash

ln -s /w205 w205

exit

```

spin up a notebook
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
```

New python3 notebook and play
```
purchases = spark.read.parquet('/tmp/purchases')

purchases.show()

purchases.registerTempTable('purchases')

purchases_by_example2 = spark.sql("select * from purchases where Host = 'user1.comcast.com'")

purchases_by_example2.show()

df = purchases_by_example2.toPandas()

df.describe()
```

down
```
docker-compose down
```
