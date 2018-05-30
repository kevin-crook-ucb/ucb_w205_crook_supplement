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



Connect to the mids container

    docker-compose exec mids bash

## At the prompt, run 

    ipython

## Try out redis

    import redis
    r = redis.Redis(host='redis', port='6379')
    r.keys()
    exit

## Exit that container

    exit

## Tear down your stack

    docker-compose down

## Verify 

    docker-compose ps

#
## Jupyter Notebooks


## Change the `docker-compose.yml` file 

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

::: notes
- Add a port for the `mids` service
- Expose adds ports
- Ports exposes it out to the host
- Port is... channel for services to talk to each other.
:::

## Save that and bring it up

    docker-compose up -d

## Start up a notebook

    docker-compose exec mids jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root

::: notes
remember, you can copy/paste this from the `course-content` repo
:::

## Copy token... should look something like

    open http://0.0.0.0:8888/?token=<your token>

## Open a browser

    http://0.0.0.0:8888

## Paste token

## Drop the cluster when you're done

    docker-compose down

#
## Automate notebook startup

##

Just for fun,

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
      - ~/w205:/w205


## 

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
          - ~/w205:/w205
        expose:
          - "8888"
        ports:
          - "8888:8888"
        extra_hosts:
          - "moby:127.0.0.1"
        command: jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root


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
