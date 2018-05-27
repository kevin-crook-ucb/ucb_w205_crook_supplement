# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


__________________________________________________________________________________________

## Assignment 05 - Employ notebooks to synthesize query project results



### Get Going


```
docker run -it --rm -p 8888:8888 -v ~/w205:/w205 midsw205/base bash
```

```
jupyter notebook --no-browser --port 8888 --ip=0.0.0.0 --allow-root 
```



#### Run queries in the notebook 

```
! bq query --use_legacy_sql=FALSE '<your-query-here>'
```

- NOTE: 
- Queries that return over 16K rows will not run this way, 
- Run groupbys etc in the bq web interface and save that as a table in BQ. 
- Query those tables the same way as in `05-assignment-example.ipynb`


#### Report
- Short description of findings and recommendations 
- If needed, add data visualizations to support recommendations 

## Resource: see example .ipynb file 
  `05-assignment-example.ipynb`
