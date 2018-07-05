```
Tonight in my office hours we worked through some examples of unrolling the assessments json file
There are parts that Spark cannot infer schema for 
specifically, if there is a mix of a list and other non-list items it wasn't able to infer schema
there are several ways to fix this:
* write python code outside of spark to fix it up before loading into spark (but it would not be MPP)
* write custom lambda transforms to fix it up in the DataFrame or RDD and then impose schema (very hard work)
* write separate lambda transforms to create separate Data Frames, register them also as temp tables, and then use joins in SparkSQL (that is the method I'll show some examples of)
```

```
Let's start with what works:

raw_assessments = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 

raw_assessments.cache()

assessments = raw_assessments.select(raw_assessments.value.cast('string'))

from pyspark.sql import Row

extracted_assessments = assessments.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()

extracted_assessments.registerTempTable('assessments')

spark.sql("select keen_id from assessments limit 10").show()

spark.sql("select keen_timestamp, sequences.questions[0].user_incomplete from assessments limit 10").show()
```

``` 
Let's talk about nulls
Spark allows some flexibility in inferring schema for json in case some of the json object have a value and others don't 
it infers null for those
here is an example of an obviously made up column and it infers it as null
try it:

spark.sql("select sequences.abc123 from assessments limit 10").show()
```

```
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
```

```
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
```


