# Suggestions for Assignment 7 for MIDS W205 2018 Summer - Kevin Crook

Part of the Berkeley culture is to not place any limits on student success.  In keeping with this culture, there are open ended components of this assignment to allow students to synthesize what they learned in the assignment to take it to higher levels.  

I would like to keep it as open ended as possible, however, I understand students have skipped some of the minimum steps in the past, so I wanted to provide a checklist of minimum components. I'm also providing a list of suggested enhancements.

## Minimum components

The following are the minimum components that I would be looking for in terms of a 9 (A):

* Create and move into directories as needed.

* Include the yml file as a separate file checked into GitHub in addition to the annotations file. 

* Download the json file of assessments.  This will be a different json file from what we did in class.  Be sure you use and annotate the correct file and make changes to all steps impacted.

* Bring up the docker cluster.

* Check the kafka logs.

* Create the kafka topic.

* Describe the kafka topic to check it.

* Explore the json file using several jq commands.

* Publish the json objects from the file to the kafka topic.

* Start pyspark
  * Subscribe to the kafka topic and store results in a spark data frame.
    * print the schema
    * show sample members of the data frame
  * Create a spark data frame to hold the json as strings
    * show sample members of the data frame
    * print the schema
    * count the number of items
    * extract individual members (2 examples)
  * Create a spark data frame to hold the json as a data frame with the same json format
    * create the new data frame
    * extract data and print
  * Exit pyspark
  
* Tear down the cluster
    
## Suggestions for annotating the minimum components:

* Everything professionally formatted in markdown.

* Annotate the architecture as defined in the yml file.  Describe each container and what it does.

* For each step, include a header, sentences to describe the step, and the step.  If the step is long, you may want to show it multi-line as I do in mine.

* Steps can occur at several levels: droplet command line, container command line, interactive python, python files, pyspark, jupyter notebook, etc.  Be sure you include all of the steps.  For example, don't just say we ran pyspark.  Give the details of what we did in pyspark and annotate. 

* For json files, pull out an individual json object and show it as an example of what the file looks like.

* For csv files, pull out the header and a couple of lines to show it as an example of what the file looks like.

## Suggestions for enhancements

Enhancements are totally open ended, so feel free to add any enhancements you wish, as I do not want to place any limits on your success.  You would need to do substantial enhancements above the minimum in terms of a 10 (A+):



