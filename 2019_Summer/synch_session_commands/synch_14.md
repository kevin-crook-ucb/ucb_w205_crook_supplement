## UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #14

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Summer/synch_session_commands/checklist_b4_class_assignments.md

### Project 3 - Understanding User Behavior Project

Assignment-09 - Define your Pipeline

Assignment-10 - Setup Pipeline, Part 1

Assignment-11 - Setup Pipeline, Part 2

Assignment-12 - Synthesis Assignment

### Full-Stack Streaming

Get Started
```
mkdir ~/w205/full-streaming-stack/

cd ~/w205/full-streaming-stack

cp ~/w205/course-content/14-Patterns-for-Data-Pipelines/docker-compose.yml .

cp ~/w205/course-content/14-Patterns-for-Data-Pipelines/*.py .
```

Full-Stack Streaming

Setup

The docker-compose.yml

Spin up the cluster
```
docker-compose up -d
```

Web-app
Take our instrumented web-app from before ```~/w205/full-streaming-stack/game_api.py```

run flask
```
docker-compose exec mids env FLASK_APP=/w205/full-streaming-stack/game_api.py flask run --host 0.0.0.0
```

Set up to watch kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```

Streaming

Run it
```
docker-compose exec spark spark-submit /w205/full-streaming-stack/write_swords_stream.py
```

Check what it wrote to Hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp

docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```

Set up Presto

Hive metastore
```
docker-compose exec cloudera hive
```

```
create external table if not exists default.sword_purchases (Accept string, Host string, User_Agent string, event_type string, timestamp string, raw_event string) stored as parquet location '/tmp/sword_purchases'  tblproperties ("parquet.compress"="SNAPPY");
```

Query this with presto
```
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```

What tables do we have in Presto?
```
show tables;
```

Describe sword_purchases table
```
describe sword_purchases;
```

Query purchases table
```
select * from sword_purchases;
```

Add some data

Seed a little data into the stream
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/

docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```

Query purchases table
```
select * from sword_purchases;
```

More data

Feed the stream more data
```
while true; do docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword; sleep 10; done
```

Watch presto grow
```
select count(*) from sword_purchases;
```

down
```
docker-compose down
```

## Building Docker Images

Setup
```
mkdir -p ~/w205/docker/mytools

cd ~/w205/docker/mytools
```

The Dockerfile
Save this as Dockerfile in ```~/w205/docker/mytools/```
```Dockerfile
FROM ubuntu:xenial
MAINTAINER Mark Mims <mark@digitalocean.com>

RUN apt-get -qq update \
  && apt-get -qq install -y jq apache2-utils
```

Build
```
docker build -t <tag> <path>
```

so, from a folder containing a Dockerfile,
```
docker build -t mytools .
```

check build ids and tags
```
docker images | grep mytools
```

test a build
```
docker run -it --rm mytools bash
```

then at the prompt
```
which jq
```

What did we do?
```
docker run -it --rm ubuntu:xenial which jq

docker run -it --rm mytools which jq
```

Iterate

You can do more in a Dockerfile
```Dockerfile
FROM ubuntu:16.04
MAINTAINER Mark Mims <mark@digitalocean.com>

ENV SPARK_VERSION        2.2.0
ENV SPARK_HADOOP_VERSION 2.6

ENV SPARK_HOME /spark-$SPARK_VERSION-bin-hadoop$SPARK_HADOOP_VERSION
ENV JAVA_HOME  /usr/lib/jvm/java-8-oracle

ENV SPARK_TEMPLATE_PATH $SPARK_HOME/templates
ENV SPARK_CONF_PATH $SPARK_HOME/conf

ENV PATH $SPARK_HOME/bin:$PATH

RUN echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections \
  && apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y software-properties-common \
  && add-apt-repository -y ppa:webupd8team/java \
  && apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF \
  && apt-get update \
  && apt-get install -y \
      curl \
      dnsutils \
      oracle-java8-installer \
  && apt-get purge -y software-properties-common \
  && apt-get autoremove -y \
  && curl -OL http://www-us.apache.org/dist/spark/spark-$SPARK_VERSION/spark-$SPARK_VERSION-bin-hadoop$SPARK_HADOOP_VERSION.tgz \
  && tar xf spark-$SPARK_VERSION-bin-hadoop$SPARK_HADOOP_VERSION.tgz \
  && rm spark-$SPARK_VERSION-bin-hadoop$SPARK_HADOOP_VERSION.tgz

COPY *-site.xml            $SPARK_TEMPLATE_PATH/
COPY *.properties          $SPARK_CONF_PATH/
COPY spark-defaults.conf   $SPARK_CONF_PATH
COPY spark-env.sh          $SPARK_CONF_PATH

COPY jars/* $SPARK_HOME/jars/

WORKDIR $SPARK_HOME

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["spark-shell"]
```

Examples of different Dockerfiles

- [Docker official images](https://github.com/docker-library/)
