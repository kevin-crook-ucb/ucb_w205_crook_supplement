## UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #12

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Before class

* VM running
* create 4 linux command lines: cluster commands, flask web API server, kafkacat, jupyter notebook server for pyspark kernel
* go through the checklist:

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Fall/synch_session_commands/checklist_b4_class_assignments.md


### Project 3 - Understanding User Behavior Project

We will wait and discuss project 3 at the end of class.


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

Write Events
`~/w205/full-stack/filtered_writes.py`

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

#### Using ssh to login without a password

**Note: this is optional.  Please be sure you have backed up any work into your GitHub remote repos.  Please follow all steps very carefully.  If you make a mistake, it's possible you will not be able to login to your VM again!**

** It looks like the Google Cloud has issues with supporting this, as they have their own scheme known as OSLogin.  OSLogin is incompatible with our VM image, so we will revisit this later this semester **


**Mac**

```
cd ~/.ssh
```

Generate a pair of keys:
```
ssh-keygen -t rsa -b 4096
(hit return through the prompts)
```

This will create two files: 
* id_rsa - the private key file
* id_rsa.pub - the public key file

Mac users will use the scp commands to copy the public key file from their local machine to the  

```
scp username@external_ip_address:/home/username/.ssh/id_rsa ~/.ssh/w205.rsa
```

Specify the private key file on the command line when connecting:
```
ssh -i ~/.ssh/w205.rsa science@ip_address
```



In your virtual machine, change to the ~/.ssh directory:

```
cd ~/.ssh
```

Generate a pair of keys:
```
ssh-keygen -t rsa -b 4096
(hit return through the prompts)
```

This will create two files: 
* id_rsa - the private key file
* id_rsa.pub - the public key file

Append the public key file to the end of the authorized_keys file:

```
cat id_rsa.pub >>authorized_keys
```

**Windows**

Windows users will first need to install two program families.  These will not use the windows installer, they will both be installed by making a directory, downloading a zip file, and extracting the zip file into the directory.

Install PuTTY:

Make a folder C:\PuTTY

Download the **Alternative binary files putty.zip (scroll down until you find it!)** in zip format using the link below, and extract the zip file into the folder c:\PuTTY

https://www.chiark.greenend.org.uk/~sgtatham/putty/

Install WinSCP:

Make a folder C:\WinSCP

Download the **portable executable (scroll down until you find it!)**  in zip format using the link below, and extract the zip file into the folder C:\WinSCP

https://winscp.net/eng/downloads.php

Using C:\WinSCP\WinSCP.exe, download the id_rsa file to your local machine (remember where you downloaded it to!).

Using C:\PuTTY\PUTTYGEN.EXE, convert the id_rsa key to putty key format.




