# Work in progress - please wait for it to be finalized


# Kevin Crook's week 11 synchronous session supplemental notes

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
    * docker pull midsw205/cdh-minimal:latest
    * docker pull midsw205/spark-python:0.0.5
    * docker pull midsw205/base:latest
* Some misc loose ends
  * Discuss having a Data Science Portfolio of your work
    * GitHub public repo
    * README.md should be informative with links to your resume in pdf, outline of areas and example
    * Consider directories such as Resume, Machine Learning, Deep Learning, Natural Language Processing, Data Visualization, Blockchain Analytics
    * Suggestion: take assignment 5, the Jupyter Notebook which queries Google Big Query and do some analytics on the Bitcoin dataset and make it look professional using markdown cells, pandas tables, data visualizations, etc.
  * Using Jupyter Notebook with Spark from Chrome on your laptop to connect to your droplet => docker cluster => spark container
  * Using a web browser from your laptop to connect to our flask web server and issue API commands
  * Using telnet from the droplet to connect to our flask web server and issue API commands
  * Using telnet from your laptop to connect to our flask web server and issue API commands
  * Using PuTTY from your laptop to connect to our flask web server and issue API commands
Activity 
  * Purpose: So far in past weeks, we have 
  
  
