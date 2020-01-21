### Project 1 Notes

Let's consider trip duration times. Run the query below and see the first few pages and last few.  What do you notice?

```sql
#standardSQL
SELECT duration_sec
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by duration_sec asc
```

The durations are given in terms of seconds.  It might be more useful to see them in terms of minutes or maybe even in hours.

```sql
#standardSQL
SELECT duration_sec, 
       cast(round(duration_sec / 60.0) as INT64) as duration_minutes,
       cast(round(duration_sec / 3600.0) as INT64) as duration_hours_rounded,
       round(duration_sec / 3600.0, 1) as duration_hours_tenths
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
order by duration_sec asc
```

