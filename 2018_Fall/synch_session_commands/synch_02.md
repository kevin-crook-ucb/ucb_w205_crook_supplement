### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #2

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md

#### Query Project

Go through the assignments 2, 3, 4

Note the same header on all 3 assignments.  The header is for the project as a whole.

#### Google BigQuery Queries

Example first query:
```sql
#standardSQL
SELECT *
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many events are there?
```sql
#standardSQL
SELECT count(*)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many stations are there?
```sql
#standardSQL
SELECT count(distinct station_id)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How long a time period do these data cover?
```sql
#standardSQL
SELECT min(time), max(time)
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

How many bikes does station 90 have (hint: total bikes should be docks_available + bikes_available)?

Does this query give us the answer?
```sql
#standardSQL
SELECT station_id, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
WHERE station_id = 90
```

No, it's time dependent.  So, let's try this query:
```sql
#standardSQL
SELECT station_id, docks_available, bikes_available, time, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
WHERE station_id = 90
ORDER BY total_bikes
```

## Create our own private dataset named bike_trip_data, create our own private table named total_bikes in our private dataset, run some queries against our private table

In the Google BigQuery user interface, on the left side panel, you will see the name of your project.  To the right of the project name, you will see a dropdown arrow.  Click on the dropdown arrow and choose "Create new dataset" and use this to create a new dataset named bike_trip_data.

Execute the following query.  Once the results come back, towards the top right of the results panel, choose "Save as Table".  Create a table named total_bikes in and put it in the dataset you just created.

```sql
#standardSQL
SELECT station_id, docks_available, bikes_available, time, 
(docks_available + bikes_available) as total_bikes
FROM `bigquery-public-data.san_francisco.bikeshare_status`
```

Using the GUI examine the new table you created going through all of the tabs.  Pay close attention to the naming and use it to create a similar queries to the ones below.

```sql
#standardSQL
SELECT distinct (station_id), total_bikes
FROM `xxxx.bike_trip_data.total_bikes`
```

```sql
#standardSQL
SELECT distinct station_id, total_bikes
FROM `xxxx.bike_trip_data.total_bikes`
WHERE station_id = 22
```

## ssh - Secure Shell

## scp - Secure Copy

Secure copy is a way to copy files from your laptop to your droplet and vice versa.  

**Windows**

Windows users will probably want to use a utility such as WinSCP 

<https://winscp.net/eng/index.php>

**Mac**

Mac users will use the command line utility scp. Here are a couple of basic examples (there are numerous variations on how to use scp):

Copy a file from your local directory to your droplet
```
scp my_file_local.txt science@ip_address:/home/science/my_file_host.txt
```

Copy a file in your droplet to your local directory
```
scp science@ip_address:/home/science/my_filehost.txt ~/my_file_local.txt
```

#### Using ssh to login without a password

In your droplet, change to the ~/.ssh directory:

```
cd ~/.ssh
```

Generate a pair of keys:
```
ssh-keygen -t rsa -b 2048
(hit return through the prompts)
```

This will create two files: 
* id_rsa - the private key file
* id_rsa.pub - the public key file

Append the public key file to the end of the authorized_keys file:

```
cat id_rsa.pub >>authorized_keys
```

**Windows**

Windows users will use WinSCP to copy the private key file down to their laptop. Use the puttygen utility to translate it into putty key file format.  When starting PuTTY, there is an option in the left panel: Connection => SSH => Auth click the Browse button to set the private key file.

**Mac**

Mac users will use the scp commands to copy the private key file down to theirlocal machine.  It usually best to rename it (such as ucb_205.rsa) and place it in the ~/.ssh directory.  

Change the mode of the file:

```
chmod 600 ~/.ssh/ucb_205.rsa 
```

Specify the private key file on the command line when connecting:
```
ssh -i ~/.ssh/ucb_205.rsa science@ip_address
```



