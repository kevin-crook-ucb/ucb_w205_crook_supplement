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

Stage and commit these file.  push the assignment branch.

When you are finished, create a pull request comparing the assignment branch to the master branch.

### docker-compose.yml

The final version will be week 13 with modifications to allow a jupyter notebook to be run in the spark container.

### game_api.py

Start with the week 11 version of game_api.py.  

You will need to add events for:
* buy a sword
* join a guild

After week 12, you will need to add metadata (additional key / value slots to the json) for each of the above, such as:

* type of sword
* guild name

### Project_3.ipynb (or similar name of your choosing)

* You will need to follow the 3 step process from my project 2 notes to get a jupyter notebook server running with a pyspark kernel
* Linux commands will be placed in markdown cells
* The pyspark code will be pulled from the spark batch jobs and modified and placed in code cells and executed
* Hive command to create schema on read placed in a markdown cell
* Presto query placed in a markdown cell

#### After week 11's synch

Create a markdown cell in the jupyter notebook and include the following linux commands:
* startup the cluster
* create the topic
* startup the flask server (may need to modify the directory and file name)
* shutdown the cluster

From the job separate_events.py, pull the code and modify it to work with jupyter notebook.  Note that code written for batch jobs has to be modified to work with jupyter notebook with pyspark kernel.

#### After week 12's synch

Modify the flask server to add metadata to your events.  The filtered_writes.py job should now filter out the other schemas so it will not dump core and give a stack trace.

In your markdown cell of linux commands, add the commands:
* individual apache bench commands
* be sure and add apache bench commands for the new events that you added

From the job filtered_writes.py, pull and modify the code to write the data frames out to parquet format.  

From the pyspark code, pull the code to read a parquet file, register as a temp table, perform some basic sql, and convert to Pandas.

The project asks you to perform some basic analytics. The analytics is very basic. You need to formulate a couple of very simple business questions and use sql against the data frames to answer the questions.

#### For week 13's synch

In your markdown cell of linux commands, add the command:
* infinite loop to run the apache bench command

From the job write_swords_stream.py
* pull and modify the code to write hdfs files in streaming mode
* sink.awaitTermination() needs to be changed to sink.stop() to stop the stream

Create a markdown cell to hold the hive command to create an external table for schema on read.  Include your hive code to create schema on read.

Create a markdown cell to hold the presto query against the external table.  Include the query and the first few lines of the result or use a limit 5, etc.

