### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #5

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Ownership issues between science and root

Files created in your droplet will be owned by science with group science. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **droplet** when logged in as science to change the owner to science and the group to science, recursively, for a directory:
```
sudo chown -R science:science w205
```

#### It's a good idea to always update the course-content repo prior to class
Note that if you made changes to your course-content repo, you won't be able to update it due to conflicts.  In that case, you will need to delete it and bring it down fresh.
```
cd ~/w205/course-content
git pull --all
cd
```

#### Before class you may want to bring down the docker images we will be using.  Some of them can take a while.
```
docker pull redis
```

#### Redis

```
docker run redis
(control c to exit)
docker ps -a
docker rm -f xxxxx

docker run -d redis
docker ps -a
docker rm -f xxxxx

docker run -d --name redis redis
docker ps -a
docker rm -f redis

docker run -d --name redis -p 6379:6379 redis
docker ps -a
docker rm -f redis
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







