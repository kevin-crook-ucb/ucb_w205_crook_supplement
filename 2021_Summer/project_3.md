## Suggestions and checklist for working on Project 3

### Project 3 repo

Clone down project 3 repo

Create an assignment branch

Work only on the assignment branch

Leave the master branch untouched

Include the following files:

* docker-compose.yml
* game-api.py
* Project_3.ipynb (or similar name - they allowed you to pick a meaningful name)

Stage and commit these file.  Push the assignment branch.

When you are finished, create a pull request comparing the assignment branch to the master branch.

### Weeks 11, 12, and 13

The assignment covers weeks 11, 12, and 13.  However, since week 12 repeats what we did in week 11 and adds to it, you will probably find it easiest to start with the week 12 material.

### AFTER week 12's synch

Copy in the week 12 docker-compose.yml file. Verify and modify if necessary to allow a Jupyter Notebook to be run against a pySpark kernel.

Startup the cluster.

Create the topic events.

Copy in the game_api.py file from week 12.  They ask you to modify it to include additional events: "buy a sword" and "join guild".  They also ask you to add metadata for each event, such as "sword type" or "guild name".

Startup the flask server.

Create a Jupyter Notebook against a pySpark kernel.  Be sure and first create the symbolic link so you can save your notebook to the project 2 repo directory.

As before:  In the Jupyter Notebook, create markdown titles, introduction, executive summary, summary, etc.  There is no set format. Just use whatever format you can come up with that you feel best conveys that you understand the pipeline, the linux commands, the Spark code, etc.

Run the Apache Bench commands.  Be sure and add new Apache Bench commands for the new events: "buy a sword" and "join guild".

From the job filtered_writes.py, pull and modify the code to write the data frames out to parquet format.  Note that code written for batch jobs has to be modified to work with jupyter notebook with pyspark kernel.

From the pyspark code, pull the code to read a parquet file, register as a temp table, perform some basic sql, and convert to Pandas.

Be sure to include a markdown cell (or cells) in the jupyter notebook:
* titles, introduction, executive summary, summary, etc. format is open ended. whatever you feel is the best format
* startup the cluster
* create the topic
* startup the flask server (may need to modify the directory and file name)
* run the apache bench commands (be sure and include for the new events)
* shutdown the cluster

### AFTER week 13's synch

Copy in the week 13 docker-compose.yml file. Verify and modify if necessary to allow a Jupyter Notebook to be run against a pySpark kernel.

Startup the cluster.

Create the topic events.

Startup the flask server.

Create a Jupyter Notebook against a pySpark kernel.  Be sure and first create the symbolic link so you can save your notebook to the project 2 repo directory.

Run the bash shell infinite loop code.  This will put a steady stream of "purchase sword" events on kafka.

Add to your Jupyter Notebook the new code from the spark submit job write_sword_stream.py  For the code sink.awaitTermination(), this is needed in the spark submit job, but not needed in the Jupyter Notebook, as code cells don't "fall off the end" like code in python file does.  If you want to stop the batch job while it's running, you can create a cell with sink.stop() and execute it to stop the stream processing.

Startup hive and run a hive command to create an external table for the schema on read. 

Starup presto and run a presto command to query the external table using the schema on read created in hive above.  If the query returns a large number of rows, you can simply use a limit 5 or similar.

The project asks you to perform some basic analytics. The analytics is very basic. You need to formulate a couple of very simple business questions and use sql against the data frames to answer the questions.

In your markdown cell of linux commands, be sure to add the command:
* infinite loop to run the apache bench command for purchase a sword
* hive command to create an external table for the schema on read
* presto query against the external table.  Include the query and first few lines of the result (you can simply use a limit 5 or similar)
* simple analytics: business questions and sql and results to answer the questions

# Checklist

### docker-compose.yml

The final version will be week 13 with modifications to allow a jupyter notebook to be run in the spark container.

### game_api.py

You will need to add events for:
* buy a sword
* join a guild

You will need to add metadata (additional key / value slots to the json) for each of the above, such as:

* type of sword
* guild name

### Project_3.ipynb (or similar name of your choosing)

markdown cells for the following:

* titles, introduction, executive summary, summary, etc.  format is open ended. whatever you feel is the best format.
* startup the cluster
* create the topic
* startup the flask server (may need to modify the directory and file name)
* run the apache bench commands (be sure and include for the new events)
* infinite loop to run the apache bench command for purchase a sword
* hive command to create an external table for the schema on read
* presto query against the external table. Include the query and first few lines of the result (you can simply use a limit 5 or similar)
* simple analytics: business questions and sql and results to answer the questions
* shutdown the cluster

code cells for the following:

* read json data from kafka
* create a spark dataframe containing the extracted json data 
* filter out only the purchase sword extracted events
* write the purchase sword extracted events to parquet files stored in hdfs
* read the parquet files from hdfs into another spark dataframe
* register the spark dataframe as a temporary table to allow in memory sql against the dataframe
* copy the dataframe to Pandas to demonstrate mixing MMP code with serial code
* read live streaming data and write it to hadoop hdfs in parquet format



