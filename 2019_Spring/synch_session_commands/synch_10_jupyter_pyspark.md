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

You will get the usual URL for Jupyter Notebook with 0.0.0.0 for the host / ip address. Since we will be connecting from Chrome on your laptop / desktop, you will need to change 0.0.0.0 with the ip address for your droplet.

Open a Chrome browser window and paste in the URL with modified ip address and the Jupyter Notebook should come up.

Once inside the Jupyter Notebook, you will notice that the directory structure is not that of the w205 directory and that the w205 directory is not listed.  It is mounted to /w205.  One quick way to rememdy this is to simply create a symbolic link from the Jupyter Notebooks directory to the /w205 directory.

First exec a bash shell into the spark container:

```
docker-compose exec spark bash
```

Create a symbolic link from the spark directory to /w205 :

```
ln -s /w205 w205
```

Exit the container:
```
exit
```

Now you should see the w205 directory listed in the Jupyter Notebook directory structure.

As a side note, anytime I have a Jupyter Notebook directory and I'm not sure which directory I'm in, one easy way to just create an Python notebook and run some code cells to pull the current working directory.

If it's on a Linux based system, you can use the following code cell:

```
!pwd
```

The following will always work, regardless of operating system, provided the os module is avaiable (it usually is):

```
import os
os.getcwd()
```
