
## Assignment 08 - Kevin Crook supplemental notes

In synchronous class week 6, we created a docker cluster with zookeeper, kafka, spark, and mids containers.  We first went through the process of creating a topic in kafka, publishing the numbers 1 to 42 to the topic, and then consuming the numbers 1 to 42.  We then used curl to download a json file from the internet containing GitHub data.  We then went through the process of creating a topic in kafka, publishing the json data to the topic (after formatting it with jq), and then consuming the json data using Python pandas.

In assignment 6, we downloaded a json file of assessment attempts and repeated the process of creating a topic in kafka and then  publishing the json data to the topic (after formatting it with jq).  

In assignment 7, we added steps to consume the json data using two different methods: kafka-console and kafkacat -C.

In synchronous class week 7, we added a new step of consuming messages using spark via the python pyspark interface.  

For assignment 08, please repeat all of the steps we did in the synchronous class week 7 for activity 2 based on my notes, but using the assessment attempts json file instead of the GitHub json file.  

Please follow our usual process in git command line and GitHub of:
* creating a branch in the repo
* making changes to the branch, staging them, committing them, and pushing them to GitHub
* in GitHub, create a pull request with your instructor as a reviewer (remove any other default reviewers)

The following should be in your repo:

* your docker-compose.yml file
* a file named <username>-annotations.md containing all command with detailed explanations following the example given.
* the instructions mention a history file, but since some students are running Windows, this will be optional

