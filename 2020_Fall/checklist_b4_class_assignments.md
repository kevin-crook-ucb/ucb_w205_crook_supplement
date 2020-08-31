### UCB MIDS W205 - Kevin Crook's checklist before class and before doing assignments:

These are general instruction for how to do some updates and cleanup before class or before working on assignments.  These will clear up a lot of issues students have struggled with in the past.

#### Make sure you are logged in as jupyter

Make sure you are logged in to your virtual machine as user jupyter. If you are in ssh and seeing another user name (such as a variation of your first and last name), then in the upper right corner, there is a gear icon, with a drop down, click on the drop down and choose "Change Linux Username", then enter the name jupyter and click the blue change button.  Please make sure you spell jupyter right.  After you change username you can verify by doing an `ls -l` command.

#### Ownership issues

Files created in your virtual machine will be owned by the jupyter username and the jupyter group. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **virtual machine** to change the owner to jupyter and the group to jupyter, recursively, for a directory. Note that you can always run this command, it won't hurt anything if it's not needed:
```
sudo chown -R jupyter:jupyter ~/w205
```

#### It's a good idea to always update the course-content repo prior to class or before working on assignments

Before class and prior to working on projects, it's best to update the course-content repo.  Note that if you made changes to your course-content repo, you won't be able to update it due to conflicts.  In that case, you will need to delete it and bring it down fresh.
```
cd ~/w205/course-content
git pull --all
cd
```

#### It's a good idea to always check for and remove any stray containers prior to class and before working on assignments

Check for stray containers:
```
docker ps -a
```

Remove a stray container (where xxxxx is the container id (hex string) or container name):
```
docker rm -f xxxxx
```

Please be careful that you do NOT remove the following container, it's part of the image for Jupyter Lab:

```
6793591eaeab        gcr.io/inverting-proxy/agent   "/bin/sh -c '/opt/bi…"   3 hours ago         Up 3 hours                              proxy-agent
```

#### Connection Pool Warning from docker-compose

When starting a cluster, you may see the following warning message:
```
WARNING: Connection pool is full, discarding connection: localhost
```

This is related to the new Google AI Platform, Notebooks instances Virtual Machine images.  (As of February 2020, it's still in Beta, so hopefully this will get increased soon.)

Generally this warning is harmless, as will just discard connections that are no longer in use.  

If you want to clean this up prior to starting a cluster, you can use these commands. 

To see the docker networks:
```
docker network ls
```

As of February 2020, these are the networks used by the Google AI Platform.  If you don't have any containers running, nor any stray containers, you should see the similar to the following:
```
NETWORK ID          NAME                DRIVER              SCOPE
2fac6dcb9205        bridge              bridge              local
7faf4695f1bf        host                host                local
5557405dd1c5        none                null                local
```

To remove all networks not used by at least 1 container:
```
docker network prune
```

To remove a specific network:
```
docker network rm xxxx
```

It's possible that stray containers, even after forced removal, may hold on to networks. In order to get rid of these, you may need to do the following:

* verify you have no stray containers with ```docker ps -a```

* stop and restart your virtual machine

* see if there are still stray networks with ```docker network ls```

* try to stop the stray networks with ```docker network rm xxxxx```


#### It's a good idea to bring down / update docker images prior to class and prior to working on assignments

We won't be using all of these day one.  (I'll be adding these as the semester goes by):

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
