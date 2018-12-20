## Suggestions for saving your work after the GitHub repos and Digital Ocean droplets (virtual machines) go away at the end of the semester

At the end of the semester, you GitHub repos and your Digital Ocean droplets (virtual machines) will go away.

The Docker images are stored on Docker Hub and they will NOT go away, so you can re-run the clusters that we built.  

### Here is a list of suggested ways to save your work (with detailed instructions to follow):

* For Windows users, make a zip file in the droplet, use WinSCP to copy the zip file down to your Windows PC, upzip, and verify that everything looks ok

* For Mac or Linux users, make a tar ball (a tar file that is compress with gzip), use scp to copy the tar ball down to your Mac or Linux machine, extract the tar ball, and verify that everything looks ok

* On your laptop, you can download either the Windows or Mac GitHub app and clone down all of your repos.  This will of course only copy repos.  It will NOT save off the directories from the synch sessions

* Make a new private repo on GitHub, rename all of the .git directories in the w205 tree, and check the entire w205 directory into the private repo on GitHub.

### For Windows users, make a zip file in the droplet, use WinSCP to copy the zip file down to your Windows PC, upzip, and verify that everything looks ok

In your droplet, login as science, and create a zip file in the droplet using the following commands:
```
cd /home/science
zip -r w205.zip w205
```

On the Windows side:

Using WinSCP gui, copy the w205.zip file down to your Windows PC

Open a Windows file explorer, navigate to the w205.zip file, right click on w205.zip, select extract all.

Using the Windows file explorer, walk the unzipped directory structure and verify that everything looks ok

### For Mac or Linux users, make a tar ball (a tar file that is compress with gzip), use scp to copy the tar ball down to your Mac or Linux machine, extract the tar ball, and verify that everything looks ok

In your droplet, login as science, and create a tar ball file in the droplet using the following commands:
```
cd /home/science
tar cvfz w205.tgz w205
```

On the Mac (or Linux laptop / desktop side):

Using scp, copy the tar ball file down to your local machine
```
scp science@my_ip_address:/home/science/w205.tgz .

(of course, you must replace my_ip_address with your ip address)
```

Extract the tar ball:
```
tar xvf w205.tgz
```

Verify directory and files extracted properly.


### On your laptop, you can download either the Windows or Mac GitHub app and clone down all of your repos.  This will of course only copy repos.  It will NOT save off the directories from the synch sessions

This is GUI based so I won't provide instructions here.

### Make a new private repo on GitHub, rename all of the .git directories in the w205 tree, and check the entire w205 directory into the private repo on GitHub.

Using the GitHub web interface, create a new private repo.  Your academic discount gives you free private repos.

In the w205 tree, for each GitHub repo, you can delete or rename the .git file.  GitHub does not allow repos inside of repos.  

```
mv .git .git.backup
```

When you have renamed all of the .git directories, you can simply cd into the /home/science/w205 directory and check it into the private repo in GitHub.

