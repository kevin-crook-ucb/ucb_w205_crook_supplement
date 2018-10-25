### Assignment 8 notes for unrolling the nested json

The assignment 8 json file has a moderately complicated structure.  It includes nested json and in several cases the nesting is multi-valued (lists).  To access these will require writing custom lamdba transform code in spark.

Let's start by reviewing how spark handles csv and json files:

#### csv files

When using csv files, the structure is always flat with no nested structure, so it has a natrual table structure, and it's very easy to load into a spark data frame and impose a schema on read (registering as a temp table) and using spark SQL to query the spark data frame.

#### json files

When using json files, the ease of use depends on the structure.  There are three main options:

* json is flat - just as easy as csv to impose schema on read and use spark SQL to query the data frame.

* json is nested, but no multi-value (no nested lists) - we can impose schema and use our "dot notation" such as xxx.yyy.zzz when yyy is nested below xxx and zzz is nested below yyy

* json is nested and multi-value (nested lists) - We can pull out single values from the list using our dot notations and/or the [] operator,  or we can pull out all values from the list by writing a custom labmda transform, creating a another data frame, registering it as a temp table, and joining it to data frames of outer nesting layers.

#### Review the structure of the assessments json file using a separate Jupyter Notebook

The following Jupter Notebook will allow you to review the structure of the assessments json file and see how the nesting with multi-valued looks:

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/assignment_08_json.ipynb

If this does not render, this website provides an online nbviewer:

https://nbviewer.jupyter.org/

#### Examples of unrolling the assessments data

In the following example, we use our dot notation with the [] operator to pull out a single item from a list.  Note that sequences.questions is a list (multi-valued).
```python
raw_assessments = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 

raw_assessments.cache()

assessments = raw_assessments.select(raw_assessments.value.cast('string'))

from pyspark.sql import Row

extracted_assessments = assessments.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()

extracted_assessments.registerTempTable('assessments')

spark.sql("select keen_id from assessments limit 10").show()

spark.sql("select keen_timestamp, sequences.questions[0].user_incomplete from assessments limit 10").show()
```




```Let's talk about nulls
Spark allows some flexibility in inferring schema for json in case some of the json object have a value and others don't 
it infers null for those
here is an example of an obviously made up column and it infers it as null
try it:

spark.sql("select sequences.abc123 from assessments limit 10").show()```


How do we do "select sequence.id from assessments limit 10" as it won't work directly?

Let's try creating another Data Frame and impose schema on it and then join it to the assessments table we just created:

def my_lambda_sequences_id(x):
    raw_dict = json.loads(x.value)
    my_dict = {"keen_id" : raw_dict["keen_id"], "sequences_id" : raw_dict["sequences"]["id"]}
    return my_dict

my_sequences = assessments.rdd.map(my_lambda_sequences_id).toDF()

my_sequences.registerTempTable('sequences')

spark.sql("select sequences_id from sequences limit 10").show()

spark.sql("select a.keen_id, a.keen_timestamp, s.sequences_id from assessments a join sequences s on a.keen_id = s.keen_id limit 10").show()


Ok, sequence.id was kinda easy because it was "flat" 
What if we want to do a list such as questions?
We would have to loop through the list and pull out the items and flat map them to the RDD

def my_lambda_questions(x):
    raw_dict = json.loads(x.value)
    my_list = []
    my_count = 0
    for l in raw_dict["sequences"]["questions"]:
        my_count += 1
        my_dict = {"keen_id" : raw_dict["keen_id"], "my_count" : my_count, "id" : l["id"]}
        my_list.append(my_dict)
    return my_list

my_questions = assessments.rdd.flatMap(my_lambda_questions).toDF()

my_questions.registerTempTable('questions')

spark.sql("select id, my_count from questions limit 10").show()

spark.sql("select q.keen_id, a.keen_timestamp, q.id from assessments a join questions q on a.keen_id = q.keen_id limit 10").show()


Here is a link to a jupyter notebook (run it outside of spark) to help understand the structure of this json file
it will read the json file
pretty print the first object (with dictionary keys in alphabetic order to make it easier to undrestand)
recursively walk through the entire structure in alphabetic order and putting indices such as [0] [1] etc. on the embeded lists

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_summer/assignments/assignment_08.ipynb

just looking at the example, I think it's easy to see how it could not infer schema
look at the pieces that don't infer and see how they look in the recursive layout


