# under construction, please wait

# Kevin Crook's week 14 synchronous session supplemental notes

Overview of today's synch session:

* Before class in your droplet:
  * get rid of any old docker containers, unless you know you are running something:
    * docker rm -f $(docker ps -aq)
  * update course-content:
    * docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
    * cd ~/course-content
    * git pull --all
    * exit
  * update the following docker images: 
    * docker pull confluentinc/cp-zookeeper:latest
    * docker pull confluentinc/cp-kafka:latest
    * docker pull midsw205/hadoop:0.0.2
      * (instead of midsw205/cdh-minimal:latest)
    * docker pull midsw205/spark-python:0.0.6
      * (up from 0.0.5)
    * docker pull midsw205/presto:0.0.1
      * (new image)
    * docker pull midsw205/base:0.1.9
* SSH
  * Let's finish up SSH from last week.
* Activity
  * Previously in Project 3:
    * We have built a docker cluster with zookeeper, kafka, cloudera hadoop, spark w python, and mids containers.  
    * We have designed and run a web API server, generated API calls using curl on the command line, a real web browser, telnet, and PuTTY raw mode.  
    * We have created a kafka topic and written API events and supporting web logs to the kafka topic.  
    * We have used spark to read the kafka topic and filter, flatten, transform, etc. events using massively parallel processing methods.  
    * We have used spark in the pyspark python oriented command line, using the spark-submit job submission style interface, and using jupyter notebook.  
    * We have also written our spark data frames using the massively parallel processing methods out to parquet format in hadoop hdfs.  We saw overwrite errors and handled them by deleting the directory and its contents or by using another name.  We also changed our code to use the overwrite option.  
    * We have used spark SQL to query our data frame using the convenience of SQL instead of the transforms.
    * We have used kafkacat in an interactive mode where it will show events as they come through. 
    * We have used Apache Bench to automate stress testing of our web API.  
    * We saw that multiple schema types on the kafka topic would break our code, so we beefed up our code to process different schemas differently.
    * We read in the data frames that we saved out to parquet format in hdfs.  (massively parallel read) 
    * We copied a spark data frame into a Pandas data frame for more convenient processing of results.
    * For spark, we have used the interfaces: pyspark command line, Jupyter Notebook, and spark-submit
  * This week:
    * We will introduce Hive which is an SQL based data warehouse platform that runs on top of hadoop hdfs.  
    * We will use hive to create a schema on read for our parquet files stored in hdfs.
    * We will run some queries from hive against our parquet files stored in hdfs.
    * We will use python code to see another way to interact with the hive metastore.
    * We will introduce another tool, Presto, to do ad hoc queries of our parquet files in hadoop hdfs.
    * (At this point we have basically built the architecture that we used with the query project for the bike share data in the Google Big Query)
    * We will introduce Spark Streaming.  So far we have just read the entire topic on kafka and written everything to hdfs.  Now we want to define batches of 10 second intervals to write in parquet to hdfs.  We will see how immutability plays into this.

## Activity


