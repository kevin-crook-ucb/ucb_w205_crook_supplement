# Connecting Jupyter Notebook to pyspark

We have been using pyspark in command line mode.  

In synch 12, we will use Juptyer Notebook connected to a pyspark kernel.

Some students want to use it earlier, so here are the instructions that will work with synch 10.  (synch 9 doesn't use pyspark)

Instead of starting a pyspark command line, use the following command to start a Jupyter Notebook for a pyspark kernel.  In this command we set the ip address to 0.0.0.0:

Multi-line for readability:
```
docker-compose exec spark \
  env \
    PYSPARK_DRIVER_PYTHON=jupyter \
    PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' \
  pyspark
```

For convenience, the command above on 1 line (remember to leave the ip address as 0.0.0.0):
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
```

You will get the usual URL for Jupyter Notebook with 0.0.0.0 for the host / ip address. Copy this and paste it into a notepad or similar.

Since we will be connecting from a Chrome browser on your laptop / desktop, in the notepad, you will need to change 0.0.0.0 to the external ip address for your Google cloud virtual machine.  Be sure you use the External IP (not the Internal IP), and remember that it changes every time you stop and start the virtual machine

If you have been running a Jupyter Notebook to another source, there will be cookie conflicts between them. The solution is to run a new ingocnito window in the Google Chrome browser.

Open a new Google Chrome browser ingocnito window, and copy and paste the URL with the modified ip address from your notepad and the Jupyter Notebook should come up.

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

## Troubleshooting Suggestions

Make sure you are using an incognito windows in the Google Chrome browser.

Make sure you are using the current external ip address from your Google Cloud virtual machine.

Check for any port 8888 issues:

Make sure you don't have any stray containers running that might be sitting on the 8888 port. If you do have stray containers, you may have to shutdown the cluster, clear all stray containers, and then restart the cluster again.  You will loose all of your work, so it's always a good idea to follow my checklist and clean up stray containers before doing any meaningful work.

Make sure port 8888 is opened, INBOUND, on the firewall for your virtual machine in the Google Cloud.  

Make sure port 8888 is opened, OUTBOUND, on the firewall for your router on your side.  If you are using a home router, outbound traffic is generally open on all ports, it's only inbound traffic that is blocked.  If you are at work, it's very likely that all outbound traffic other than http and https will be blocked to external servers.

If you are running a VPN (virtual private network), it's very likely that port 8888 will be blocked.  Sometimes students will leave a VPN running and forget to stop it, or it may automatically start and you may have to stop it.

The easiest way to check for firewall or VPN issues with port 8888 is to send your URL to someone else and have them see if the URL works from their location.  If it works from their location the problem is with your firewall on your side. 

