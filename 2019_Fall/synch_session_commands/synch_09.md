### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #9

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Before class

* VM running
* create 3 linux command lines: general cluster commands, flask web API server, curl web API test commands
* go through the checklist:

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Fall/synch_session_commands/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

We will talk about project 3 at the end of class.

### Flask with Kafka

```
mkdir ~/w205/flask-with-kafka

cd ~/w205/flask-with-kafka

cp ~/w205/course-content/09-Ingesting-Data/docker-compose.yml .
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
```
cp ~/w205/course-content/09-Ingesting-Data/basic_game_api.py .
```

run it via
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/basic_game_api.py flask run
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
```
cp ~/w205/course-content/09-Ingesting-Data/game_api.py .
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
