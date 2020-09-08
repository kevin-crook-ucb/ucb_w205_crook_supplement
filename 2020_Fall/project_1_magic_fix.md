### Fix for using the Google BigQuery magic commands

In August 2020, Google released an update to their AI Platform.  That release appears to have broken the magic commands in Jupyter Notebook for Google BigQuery.

Here is a temporary workaround that you can you until Google patches their release with a fix:

Create a initial cell in the Jupyter Notebook with the following and execute that cell:

```
import sys
!{sys.executable} -m pip install -U google-cloud-bigquery[bqstorage,pandas]
```
