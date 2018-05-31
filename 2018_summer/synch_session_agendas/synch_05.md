# under construction

# UCB MIDS W205 Summer 2018 - Kevin Crook's agenda for Synchronous Session #5

## Update docker images (before class)

Run these command in your droplet (but **NOT** in a docker container):

```
docker pull midsw205/base:latest
docker pull redis:latest
docker pull confluentinc/cp-zookeeper:latest
docker pull confluentinc/cp-kafka:latest
```

## Update the course-content repo in your docker container in your droplet (before class)

See instructions in previous synchronous sessions.

## Create a docker cluster with a container for redis (key-value data store) and a container for mids base ubuntu 

Outside of docker, create a directory called redis that will hold our yml file for this docker cluster:

```
mkdir ~/w205/redis
```

Change to redis directory:

```
cd ~/w205/redis
```

Use vi to create a docker-compose.yml file with the content below.

```
vi docker-compose.yml
```

```yml
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    extra_hosts:
      - "moby:127.0.0.1"
```

Startup the docker cluster:

```
docker-compose up -d
```

Verify the docker cluster is running properly:

```
docker-compose ps
```

Output should be similar to this:

```
             Name                        Command               State    Ports     
    ---------------------------------------------------------------------------
    redisexample_midsbase_1   /bin/bash                        Up      8888/tcp 
    redisexample_redis_1      docker-entrypoint.sh redis ...   Up      6379/tcp 
```

Look at the logs of the redis container in the cluster:

```
docker-compose logs redis
```

The last of the output should be similar to this to let us know that redis it ready to accept connections:

```
Ready to accept connections
```

We are running our cluster containers "headless", that is without an interactive connection.  What if we want to interact with a container, such as running a bash shell with standard input and standard output piped to and from our command prompt?  We can exec processes in the containter, including bash shells:

```
docker-compose exec mids bash
```

We now have a bash shell running in our mids container inside our docker cluster.

We now want to run ipython (interactive python) which is a command line utility similar to jupyter notebook.  It actually predates jupyter notebook and was the basis on which jupter notebooks was designed and built.

```
ipython
```

In our ipython command line, we will try out the redis database.  Redis is a key-value pair, NoSQL database.  Check TCP port 6379 against the specification in the yml file.

```
import redis
r = redis.Redis(host='redis', port='6379')
r.keys()
exit
```

Exit our bash shell which was exec'd in the container.

```
exit
```

Tear down our docker cluster

```
docker-compose down
```

Verify the docker cluster is down

```
docker-compose ps
```

```
docker ps -a
```

## Adding Jupyter Notebooks to the mids container in our docker cluster

We will change our docker-compose.yml file to allow jupyter notebooks to be run in the mids container.  We will expose the correct ports to the outside world so we can connect from a Chrome browser on our laptops to jupyter notebook running in our mids container in our droplet.

Using vi, edit our docker-compose.yml file to match the following:

```yml
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"
```

Bring up our docker cluster, "detached" or "headless".

```
docker-compose up -d
```

Exec a jupyter notebook in our mids container in our docker cluster.  Compare port 888 to our specfication in the yml file.

```
docker-compose exec mids jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```

On your laptop, open a Chrome browser and connect to jupyter notebook running in the mids container of your docker cluster in your droplet.  Please note that outgoing TCP port 8888 must not be blocked between your laptop and your droplet. 

```
http://xxx.xxx.xxx.xxx:8888/?token=xxx
```

Exit and logout of jupyter notebook in your browser. 

Tear down your docker cluster.

```
docker-compose down
```

Verify the docker cluster is down.  (This is the last time I'll put verify instruction from now on, always verify that a docker cluster tears down properly)

```
docker-compose ps
```

```
docker ps -a
```

## Breakout - students discuss Query Project

We will go into breakout for students to discuss the Query Project.  If students are having trouble with getting their cluster from the previous to work, or their jupyter notebook to work, the instructor can work with them one-on-one to get caught up.


## Automatically start Jupyter Notebook in the mids container of your docker cluster

Let's change our mids container to automatically start the jupyter notebook so we don't have to manually start it each time.

krc - stopping point


#
## Automate notebook startup

##

Just for fun,

```yml
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"
    command: jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```


## Test it out

    docker-compose up -d

## Run to get the token 

    docker-compose logs mids

## Open a browser

    open http://0.0.0.0:8888/?token=<your token>

## Open New Python3 Notebook

## Try redis

    import redis
    r = redis.Redis(host='redis', port='6379')
    r.keys()
    
## Add some values

    r.set('foo', 'bar')
    value = r.get('foo')
    print(value)

## Drop cluster

    docker-compose down


#
## Redis to track state

::: notes
See nosql-kv-stores-video-hd1080-h264-30fps.mp4 
:::

## Setup

Download data:

    cd ~/w205/
    curl -L -o trips.csv https://goo.gl/MVNVhW

## Setup

Add volumes to your `docker-compose.yml`:

    volumes:
      - /home/science/w205:/w205

```yml
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    volumes:
      - /home/science/w205:/w205
    expose:
      - "8888"
    ports:
      - "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"
    command: jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```


## Spin up cluster

    docker-compose up -d

## Run to get the token 

    docker-compose logs mids

## Open a browser

    open http://0.0.0.0:8888/?token=<your token>

## Open New Python3 Notebook

## 

    import redis
    import pandas as pd

##

    trips=pd.read_csv('trips.csv')

    date_sorted_trips = trips.sort_values(by='end_date')

    date_sorted_trips.head()

##

    for trip in date_sorted_trips.itertuples():
      print(trip.end_date, '', trip.bike_number, '', trip.end_station_name)

::: notes
print date sorted list of where all bikes are
:::


##

    current_bike_locations = redis.Redis(host='redis', port='6379')
    current_bike_locations.keys()
    
## Add values

    for trip in date_sorted_trips.itertuples():
      current_bike_locations.set(trip.bike_number, trip.end_station_name)

##

    current_bike_locations.keys()

## Where is bike 92? 

    current_bike_locations.get('92')

## Drop cluster

    docker-compose down
