# Suggestions for Assignment 8 for MIDS W205 2018 Summer - Kevin Crook

Part of the Berkeley culture is to not place any limits on student success.  In keeping with this culture, there are open ended components of this assignment to allow students to synthesize what they learned in the assignment to take it to higher levels.  

I would like to keep it as open ended as possible, however, I understand students have skipped some of the minimum steps in the past, so I wanted to provide a checklist of minimum components. I'm also providing a list of suggested enhancements.

Also, remember that assignments 6, 7, and 8 are part of the Project 2.  You will want to reuse as much content from the previous assignment.  

## Assignment 8 relates to Synchronous 8

## Minimum components

The following are the minimum components that I would be looking for in terms of a 9 (A).  These are the raw commands from class.  They may need adaptation to work for the differing json file and to be adapted for any enhancements you make:

```
mkdir ~/w205/spark-with-kafka-and-hdfs
cd ~/w205/spark-with-kafka-and-hdfs
```
```
cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
```
```
docker-compose up -d
```
```
docker-compose logs -f kafka
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
```
docker-compose exec kafka \
  kafka-topics \
    --create \
    --topic commits \
    --partitions 1 \
    --replication-factor 1 \
    --if-not-exists \
    --zookeeper zookeeper:32181
```
```
cd ~/w205/spark-with-kafka-and-hdfs
curl -L -o github-example-large.json https://goo.gl/Hr6erG
```
```
docker-compose exec mids \
  bash -c "cat /w205/spark-with-kafka-and-hdfs/github-example-large.json \
    | jq '.[]' -c \
    | kafkacat -P -b kafka:29092 -t commits"
```
```
docker-compose exec spark pyspark
```
```
raw_commits = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","commits") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
```
```
raw_commits.cache()
```
```
raw_commits.printSchema()
```
```
commits = raw_commits.select(raw_commits.value.cast('string'))
```
```
commits.write.parquet("/tmp/commits")
```
```
extracted_commits = commits.rdd.map(lambda x: json.loads(x.value)).toDF()
```
```
extracted_commits.show()
```
```
extracted_commits.printSchema()
```
```
extracted_commits.registerTempTable('commits')
```
```
spark.sql("select commit.committer.name from commits limit 10").show()
spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10").show()
```
```
some_commit_info = spark.sql("select commit.committer.name, commit.committer.date, sha from commits limit 10")
```
```
some_commit_info.write.parquet("/tmp/some_commit_info")
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/
docker-compose exec cloudera hadoop fs -ls /tmp/commits/
```
```
exit()
```
```
docker-compose down
docker-compose ps
```

## Suggestions for annotating the minimum components:

* Everything professionally formatted in markdown.

* Annotate the architecture as defined in the yml file.  Describe each container and what it does.

* For each step, include a header, sentences to describe the step, and the step.  If the step is long, you may want to show it multi-line as I do in mine.

* Steps can occur at several levels: droplet command line, container command line, interactive python, python files, pyspark, jupyter notebook, etc.  Be sure you include all of the steps.  For example, don't just say we ran pyspark.  Give the details of what we did in pyspark and annotate. 

* For json files, pull out an individual json object and show it as an example of what the file looks like.

* For csv files, pull out the header and a couple of lines to show it as an example of what the file looks like.

## Suggestions for enhancements

**Be sure to include an Enchancement section at the end of your submission, preferably with a bullet list of enhancements where each enchance is a bullet point with a brief description.**

Assignments 6, 7, and 8 are part of Project 2.  You can reuse enhancements in the next assignemnt.

Enhancements are totally open ended, so feel free to add any enhancements you wish, as I do not want to place any limits on your success.  You would need to do substantial enhancements above the minimum in terms of a 10 (A+):

* Executive summary

* Introduction

* Architecure (some included architecture diagrams they created externally and uploaded images to include in mark down)

* Add steps to explore and understand further in additional to the minimum steps.

* We are doing the data engineering piece and handing it over to the data scientists.  What types of analytics could the data scientists do?  Speed Layer?  

* Other business problems this same technology can be used for.

* Furture enhancements that are possible but we didn't have time to build out.

* Summary of findings, wrap up.

* Appendix: List of Enhancements (be sure to include this at the end for grading purposes)
