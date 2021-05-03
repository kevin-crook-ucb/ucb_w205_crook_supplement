### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #5

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2021_Spring/checklist_b4_class_assignments.md

#### Add firewall rule to open port 8888 

In the Google Cloud, we need to add a firewall rule to allow traffic on port 8888.

Navigation Menu => VPC Network => Firewall Rules

Check under the column Protocols / ports and see if you have an entry:
```
tcp:8888
```

If you do not, on the top, click on CREATE FIREWALL RULE and fill out as follows:

Name: 
jupyter-notebook

Description:
Adds Jupyter Notebook access via port 8888

Logs:
Off

Network:
default

Priority: 
1000

Direction of traffic:
Ingress

Action on match:
Allow

Targets:
All instances in the network

Source filter:
IP ranges

Source IP ranges:
0.0.0.0/0

Second source filter:
None

Protocols and ports:
Specified protocols and ports
check tcp and put 8888 in the box next to it

Enforcement:
Enabled should be selected

Click blue Create button

#### Install Docker Compose

Try the following command and see if it works:

```
docker-compose
```

If it doesn't you need to install Docker Compose using the following commands:

```
sudo apt update
sudo apt install docker-compose
```

#### Create a docker cluster with redis standalone

```
mkdir ~/w205/redis-standalone
cd ~/w205/redis-standalone
cp ../course-content/05-Storing-Data-II/example-0-docker-compose.yml docker-compose.yml
```

Review the docker-compose.yml file.

Spinup
```
docker-compose up -d
```

Check stuff
```
docker-compose ps
```

Peek at the logs
```
docker-compose logs redis
```

Should see
```
Ready to accept connections
```

Run stuff 
```
ipython
```

```python
import redis
r = redis.Redis(host='localhost', port='6379')
r.keys()
exit
```

If the redis module was not installed in the VM, use pip to install it and retry the previous python code.

```
pip install redis
```

Tear down the stack:
```
docker-compose down
```

Verify:
```
docker-compose ps
```

#### Create a second docker cluster with Redis and Mids

```
mkdir ~/w205/redis-cluster
cd ~/w205/redis-cluster
cp ../course-content/05-Storing-Data-II/example-1-docker-compose.yml docker-compose.yml
```

Startup the cluster
```
docker-compose up -d
```

Check stuff
```
docker-compose ps
```

Peek at the logs
```
docker-compose logs redis
```

Run stuff
```
docker-compose exec mids bash
```

At the prompt, run
```
ipython
```

Try out redis
```python
import redis
r = redis.Redis(host='redis', port='6379')
r.keys()
exit
```

Exit that container
```
exit
```

Tear down your stack
```
docker-compose down
```

Verify
```
docker-compose ps
```

#### Create a third docker cluster with Redis and Mids supporting Jupyter Notebook on port 8888

Change the docker-compose.yml file
```
cp ../course-content/05-Storing-Data-II/example-2-docker-compose.yml docker-compose.yml
```

Bring it up
```
docker-compose up -d
```

Start a notebook (leave 0.0.0.0 as is, that is the internal facing network ip)
```
docker-compose exec mids jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```

Copy the Jupyter Notebook http string, change the ip address to that of your droplet (same ip you use to login to your droplet), open a browser, surf

Drop the cluster when you're done
```
docker-compose down
```

#### Automate notebook startup

Change the docker-compose.yml file
```
cp ../course-content/05-Storing-Data-II/example-3-docker-compose.yml docker-compose.yml
```

test it out
```
docker-compose up -d
```

Run to get the token
```
docker-compose logs mids
```

Copy the Jupyter Notebook http string, change the ip address to that of your droplet (same ip you use to login to your droplet), open a browser, surf

Open new Python3 notebook

Try redis
```python
import redis
r = redis.Redis(host='redis', port='6379')
r.keys()
```

Add some values
```
r.set('foo', 'bar')
value = r.get('foo')
print(value)
```

Drop cluster
```
docker-compose down
```

#### Redis to track state

Change the docker-compose.yml file
```
cp ../course-content/05-Storing-Data-II/example-4-docker-compose.yml docker-compose.yml
```

Download data
```
cd ~/w205/
curl -L -o trips.csv https://goo.gl/QvHLKe
```

Spin up the cluster
```
cd ~/w205/redis-cluster
docker-compose up -d
```

Run to get the token
```
docker-compose logs mids
```

Copy the Jupyter Notebook http string, change the ip address to that of your droplet (same ip you use to login to your droplet), open a browser, surf

Open new Python3 notebook
```python
import redis
import pandas as pd

trips=pd.read_csv('trips.csv')

date_sorted_trips = trips.sort_values(by='end_date')

date_sorted_trips.head()

for trip in date_sorted_trips.itertuples():
      print(trip.end_date, '', trip.bike_number, '', trip.end_station_name)

current_bike_locations = redis.Redis(host='redis', port='6379')

current_bike_locations.keys()

for trip in date_sorted_trips.itertuples():
      current_bike_locations.set(trip.bike_number, trip.end_station_name)
      
current_bike_locations.keys()
```

Where is bike 92?
```python
current_bike_locations.get('92')
```

Drop cluster
```
docker-compose down
```
