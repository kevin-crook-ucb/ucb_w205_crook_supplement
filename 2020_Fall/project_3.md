## Checklist for working on Project 3

### Project 3 repo

Clone down project 3 repo

Create an assignment branch

Work only on the assignment branch

Leave the master branch untouched

Include the following files:

* docker-compose.yml
* game-api.py
* Project_3.ipynb (or similar name - they allowed you to pick a meaningful name)

### docker-compose.yml

2) docker-compose.yml - the final version will be week 13 with modifications to allow a jupyter notebook server to be run in the spark container.

3) game-api.py - the final version will be week 13, but modifications from prior weeks are minimal.  

3a) you will need to add events for: buy a sword and join a guild

3b) you will need to add at least 1 additional metadata for each event.  they suggest sword type and guild name

4) jupyter notebook, such as Project_3.ipynb, including the following code which you can pull out and modify from the spark batch jobs:

4a) read json data from kafka

4b) create a spark dataframe containing the extracted json data

4c) filter out only the purchase a sword extracted events

4d) write the purchase sword extracted events to parquet files stored as hdfs

