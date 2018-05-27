---
title: Fundamentals of Data Engineering
author: Week 04 - sync session
...

---


#
## While we're getting started

- Review your Project Questions 
- Get ready to share

::: notes
Breakout at about 5 after the hour:
- Check in with each group on their solution to the assignment
- Answer questions as people have them on what they had trouble with
- Usually takes 10-20 minutes
:::

#
## Overview
- Go over Assignment 3 results
- PRs from the command line
- Docker compose, beginning


# 

## Assignment 3: Our Project Questions
- What is a trip?
- What are the most common trips?
- How does this differ based on trip type (commuter vs all)?
- What is a commuter trip?

::: notes
- What is a commuter trip?
  * A trip during rush hour
  * Write a query to determine if a trip happens from 7-9 am or 4-7 pm. (or how many trips do...)
:::


## Due Friday (PR)




## Housekeeping

- Class flow

:::notes
- The following few slides review the flow of when things are due and what readings/videos go with which week.
:::

## Between Class 3 & Class 4

> - async material in Week 3 syllabus
> - Readings in Week 3 syllabus
> - Assignment 03
> - Final Assignment 02 was due last Friday


## Class 4 (i.e., today)

> - Groupwork & present Assignment 03
> - Activities Week 3 async topic
> - Review Query Project (spans Assignments 2-5)
> - Activities for Assignment 04 (refining and answering your project questions)

## Between Class 4 & Class 5

> - async material in Week 4 syllabus (RDBs & NoSQL)
> - Readings in Week 4 syllabus
> - Assignment 04
> - Final Assignment 03 due on Friday


## Where are we in the Query Project?

- Assignment 4: Answer your Project Questions
- Assignment 5: Use Jupyter Notebook to do some visualizations and write up your reasoning for the recommendations you choose to make.

::: notes
use bigquery or bq cli for assignment 4
:::


# 

## Creating a GitHub Pull-Request(PR)
- From the command-line

## Clone a repo from GitHub

```
git clone \
  https://github.com/mids-w205-martin-mims/assignment-02-htmartin
```

    cd assignment-02-htmartin

## Create a branch to work from

    git branch my-cool-feature

## Switch to that branch

    git checkout my-cool-feature

## Make changes to code

    vi README.md

::: notes
- M: no vi in container? droplets?
:::

## Commit those changes

    git commit -m'updated README' README.md

## Push those up to GitHub

    git push origin my-cool-feature


## Note  
- If this is the first time you've pushed to the remote `my-cool-feature` branch, then this command will automatically _create_ that branch in your github repo and then push your changes to it.

## Pull Request from the GitHub Web-UI

- Select "New Pull-Request"

- Select branches so that you are "Requesting to merge changes from `my-cool-feature` branch _into_ `master`."

- Select your instructor(s) to review.

- Submit






#
## Docker: Where am I?

- We've worked with Docker
- We've explore cli ways to find out what's up with data
- How do we find our way around with Docker?

::: notes
docker run -it --rm -v ~/w205:/w205 midsw205/base bash

:::

## Run the regular container

```
docker run -it --rm -v ~/w205:/w205 midsw205/base bash
```

## What containers are running right now?

- New terminal window

- `docker ps`

## What containers exist?

- `docker ps -a`

## Container name

- `fervent_austin` is my running `midsw205/base:latest` container

## What images do I have?

- Images vs. containers

- `docker images`

## Image name 

- Need both repository & tag

- e.g., `midsw205/base:latest`




## Clean up containers

`docker rm -f <name-of-container>`

::: notes
:::

#
## Idiomatic docker

- start a `midsw205/base:latest` container
- run `pwd`
- exit container

-vs-

```
docker run -it --rm -v ~/w205:/w205 midsw205/base pwd
```
 



::: notes
I hope some of this simplifies when we start using the containers to _just_ run a command... i.e.,
`docker run [<opts>] <image> [<command>]`
... e.g., 
ME: check this query for backticks from bq cli sql
`docker run -it --rm midsw205/base bq query --use_legacy_sql=false 'select count(*) from mytable'`
in one go (edited)

then they're only "in" one place
:::

#
## Docker compose

- What is docker compose?

## Update your course content repo in w205

```
cd ~/w205/course-content
git pull --all
```

## Docker compose .yml file

- `cd w205`
- `mkdir kafka`
- save `docker-compose.yml` from recently pulled `~/w205/course-content` to
  recently created `~/w205/kafka` directory


::: notes

Save the following snippet as `~/w205/kafka/docker-compose.yml` on your host
filesystem

    ---
    version: '2'
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:latest
        network_mode: host
        environment:
          ZOOKEEPER_CLIENT_PORT: 32181
          ZOOKEEPER_TICK_TIME: 2000
        extra_hosts:
          - "moby:127.0.0.1"

      kafka:
        image: confluentinc/cp-kafka:latest
        network_mode: host
        depends_on:
          - zookeeper
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: localhost:32181
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:29092
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        extra_hosts:
          - "moby:127.0.0.1"

:::




## Docker compose spin things up

- `cd ~/w205/kafka`
- `docker-compose up -d`
- `docker-compose ps`

::: notes
- This is the start of spinning up things that will lead to projects 2&3
- Have them go through on command line, talk about what is happening.
:::

## Clean up

`docker-compose down`

- Can check with:
- `docker-compose ps`


#
## Summary
- git branching
- where are we with Docker?
- Idiomatic Docker
- docker-compose

##

![](images/pipeline-overall.svg)

::: notes
docker-compose is for this
:::


#

## 


::: notes
md works here
:::

# 

## Extras




#

<img class="logo" src="images/berkeley-school-of-information-logo.png"/>
