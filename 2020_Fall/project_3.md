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

Stage and commit these file.  push the assignment branch.

When you are finished, create a pull request comparing the assignment branch to the master branch.

### docker-compose.yml

The final version will be week 13 with modifications to allow a jupyter notebook to be run in the spark container.


### game-api.py

The final version will be week 13, but modifications from prior weeks are minimal.

You will need to add events for:
* buy a sword
* join a guild

You will need to add metadata (additional key / value slots to the json) for each of the above, such as:

* type of sword
* guild name

### Project_3.ipynb (or similar name of your choosing)

The pyspark code will be pulled from the spark batch jobs.  The final version will be based on the spark batch job write_swords_stream.py

Be sure you include code for the following:
*
