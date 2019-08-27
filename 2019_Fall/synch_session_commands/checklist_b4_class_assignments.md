### UCB MIDS W205 - Kevin Crook's checklist before class and before doing assignments:

These are general instruction for how to do some updates and cleanup before class or before working on assignments.  These will clear up a lot of issues students have struggled with in the past.

#### Ownership issues

Files created in your virtual machine will be owned by your Google generated username and group. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **virtual machine** to change the owner to xxxxx and the group to yyyyy, recursively, for a directory:
```
sudo chown -R xxxxx:yyyyyy ~/w205
```

#### It's a good idea to always update the course-content repo prior to class or before working on assignments
Note that if you made changes to your course-content repo, you won't be able to update it due to conflicts.  In that case, you will need to delete it and bring it down fresh.
```
cd ~/w205/course-content
git pull --all
cd
```

#### It's a good idea to always check for and remove any stray containers prior to class and before working on assignments

Check for stray containers
```
docker ps -a
```

Remove a stray container (where xxxxx is the container id (hex string) or container name)
```
docker rm -f xxxxx
```

Remove all containers.  Nuclear option will destory all containers.  Good for cases when you have several stray containers and you are sure you don't want to keep any of them
```
docker rm -f $(docker ps -aq)
```

#### It's a good idea to bring down / update docker images prior to class and prior to working on assignments

We won't be using all of these day one.  (I'll be adding these as the semester goes by)

```
docker pull midsw205/base:latest
docker pull midsw205/base:0.1.8
docker pull midsw205/base:0.1.9
docker pull redis
docker pull confluentinc/cp-zookeeper:latest
docker pull confluentinc/cp-kafka:latest
docker pull midsw205/spark-python:0.0.5
docker pull midsw205/spark-python:0.0.6
docker pull midsw205/cdh-minimal:latest
docker pull midsw205/hadoop:0.0.2
docker pull midsw205/presto:0.0.1


```
