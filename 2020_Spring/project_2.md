### Project 2 notes for unrolling the nested json  

The project 2 json file has a moderately complicated structure.  It includes nested json and in several cases the nesting is multi-valued (lists).  To access these will require writing custom lamdba transform code in spark.

Let's start by reviewing how spark handles csv and json files:

#### csv files

When using csv files, the structure is always flat with no nested structure, so it has a natrual table structure, and it's very easy to load into a spark data frame and impose a schema on read (registering as a temp table) and using spark SQL to query the spark data frame.

#### json files

When using json files, the ease of use depends on the structure.  There are some common options:

* json is flat - Just as easy as csv to impose schema on read and use spark SQL to query the data frame.

* json is nested, but no multi-value (no nested list or nested dictionary) - We can impose schema and use our "dot notation" such as xxx.yyy.zzz when yyy is nested below xxx and zzz is nested below yyy

* json is nested and multi-valued in the form of a dictionary - We need to write a custom lambda transform to extract the json dictionary string, convert it into a python dictionary, extract the data we need, create another data frame, register it as a temp table, and join it to data frames of the outer nesting layers.

* json is nested and multi-valued in the form of a list - We can pull out single values from the list using our dot notations and/or the [] operator.  We can pull out all values from the list by writing a custom labmda transform, creating a another data frame, registering it as a temp table, and joining it to data frames of outer nesting layers.

* json is nested in a complex multi-valued way - a list nests a dictionary that nests a list, etc. in various combinations.

#### Review the structure of the assessments json file using a separate Jupyter Notebook

The assessments json file is nested in a complex multi-valued way.  It has nested dictionaries that nest lists that nest dictionaries that nest lists that nest dictionaries. 

The following Jupter Notebook will allow you to review the structure of the assessments json file and see how the nesting with multi-valued looks:

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2020_Spring/project_2_json.ipynb

If this does not render, this website provides an online nbviewer:

https://nbviewer.jupyter.org/

#### Examples of unrolling the assessments data

In the following example, we use our dot notation with the [] operator to pull out a single item from a list.  Note that sequences.questions is a list (multi-valued).
```python
raw_assessments = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","assessments").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 

raw_assessments.cache()

assessments = raw_assessments.select(raw_assessments.value.cast('string'))

import json

from pyspark.sql import Row

extracted_assessments = assessments.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()

extracted_assessments.registerTempTable('assessments')

spark.sql("select keen_id from assessments limit 10").show()

spark.sql("select keen_timestamp, sequences.questions[0].user_incomplete from assessments limit 10").show()
```

Missing Values in some json objects - Spark allows some flexibility in inferring schema for json in the case of some of the json objects have a value and others don't have the value.  It infers a null for those.  Here is an example of an obviously made up column called "abc123" and see that it infers null for the column:
```python
spark.sql("select sequences.abc123 from assessments limit 10").show()
```

#### Nested multi-value as a dictionary

Let's see an example of a nested multi-value as a dictionary.  First note that the following will NOT work because sequences value is a dictionary, so id is a key of the nested dictionary:
```python
# does NOT work!
spark.sql("select sequence.id from assessments limit 10").show()
```

We can extract sequence.id by writing a custom lambda transform, creating a separate data frame, registering it as a temp table, and use spark SQL to join it to the outer nesting layer:
```python
def my_lambda_sequences_id(x):
    raw_dict = json.loads(x.value)
    my_dict = {"keen_id" : raw_dict["keen_id"], "sequences_id" : raw_dict["sequences"]["id"]}
    return Row(**my_dict)

my_sequences = assessments.rdd.map(my_lambda_sequences_id).toDF()

my_sequences.registerTempTable('sequences')

spark.sql("select sequences_id from sequences limit 10").show()

spark.sql("select a.keen_id, a.keen_timestamp, s.sequences_id from assessments a join sequences s on a.keen_id = s.keen_id limit 10").show()
```

#### Nested multi-valued as a list

Let's see an example of a multi-valued in the form of a list.  Previously, we saw that we can pull out 1 item using the [] operator. In this example, we will pull out all values from the list by writing a custom labmda transform, creating a another data frame, registering it as a temp table, and joining it to data frames of outer nesting layers.
```python
def my_lambda_questions(x):
    raw_dict = json.loads(x.value)
    my_list = []
    my_count = 0
    for l in raw_dict["sequences"]["questions"]:
        my_count += 1
        my_dict = {"keen_id" : raw_dict["keen_id"], "my_count" : my_count, "id" : l["id"]}
        my_list.append(Row(**my_dict))
    return my_list

my_questions = assessments.rdd.flatMap(my_lambda_questions).toDF()

my_questions.registerTempTable('questions')

spark.sql("select id, my_count from questions limit 10").show()

spark.sql("select q.keen_id, a.keen_timestamp, q.id from assessments a join questions q on a.keen_id = q.keen_id limit 10").show()
```

#### How to handle "holes" in json data

When unrolling the json for the assessments dataset, if you are trying to unroll a key in a dictionary that does not exist for all the items, it will generate an error when you try to reference in the cases it does not exist.

Below is some example code for raw_dict["sequences"]["counts"]["correct"] which exists for some but not all of the json objects.  To keep it from generating errors, you would need to check it piece meal to make sure it exists before referencing it. 

We could just default it to 0 if it doesn't exist.  

However, suppose we want to find the average or standard deviatation, the 0's would skew the data too low.  In order to not include it, I add a level of indirection on top of the dictionary and only add the dictionary if it has meaningful data.  Instead of using the "map()" spark functional transformation, I use the "flatMap()" functional transformation, which removes a level of indirection at the end.

Here is how the flat map works in this case.  Suppose A, B, C, and D are all dictionaries:

```( (A), (), (B), (), (), (C), (D), () )```

flat maps to:

```( A, B, C, D)```

Here is the full pyspark code:

```python
def my_lambda_correct_total(x):
    
    raw_dict = json.loads(x.value)
    my_list = []
    
    if "sequences" in raw_dict:
        
        if "counts" in raw_dict["sequences"]:
            
            if "correct" in raw_dict["sequences"]["counts"] and "total" in raw_dict["sequences"]["counts"]:
                    
                my_dict = {"correct": raw_dict["sequences"]["counts"]["correct"], 
                           "total": raw_dict["sequences"]["counts"]["total"]}
                my_list.append(Row(**my_dict))
    
    return my_list

my_correct_total = assessments.rdd.flatMap(my_lambda_correct_total).toDF()

my_correct_total.registerTempTable('ct')

spark.sql("select * from ct limit 10").show()

spark.sql("select correct / total as score from ct limit 10").show()

spark.sql("select avg(correct / total)*100 as avg_score from ct limit 10").show()

spark.sql("select stddev(correct / total) as standard_deviation from ct limit 10").show()
```
