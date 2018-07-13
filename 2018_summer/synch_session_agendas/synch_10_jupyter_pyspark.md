# Connecting Jupyter Notebook to pyspark

We have been using pyspark in command line mode.  

In synch 12, we will use Juptyer Notebook connected to a pyspark kernel.

Some students want to use it earlier, so here are the instructions that will work with synch 10.  (synch 9 doesn't use pyspark)

Instead of starting a pyspark command line, use the following command to start a Jupyter Notebook for a pyspark kernel:

Multi-line for readability:
```
docker-compose exec spark \
  env \
    PYSPARK_DRIVER_PYTHON=jupyter \
    PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' \
  pyspark
```

For convenience, the command above on 1 line:
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
```

You will get the usual URL for Jupyter Notebook with 0.0.0.0 for the host / ip address. Since we will be connecting from Chrome o
