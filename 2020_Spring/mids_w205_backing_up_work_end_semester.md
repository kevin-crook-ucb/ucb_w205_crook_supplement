# work in progress - please wait until I have tested and verified these instruciton

### MIDS w205 - Backing up your work at the end of the semester

#### GitHub classroom repos

GitHub classroom will delete all repos in the mids-w205-crook organization on GitHub.  This includes the repos for course content and projects 1, 2, and 3. 

If you forked a repo into the mids-w205-crook organization, the fork will also be deleted.

If you forked a repo in the mids-w205-crook organizatio into your own organization, I'm not sure if it will be deleted or not.  GitHub classroom does not have any documentation that seems to address this, and I don't have a way to test it.   You can fork a repo in the mids-w205-crook organization to your own organization, but I would recommend that you also make another backup of it.

#### Your VM in the GCP

Your virtual machine (VM) in the Google Cloud Platform (GCP) will not go away, unless you delete it.  

The w205 directory should have all of your work for the semester, including course content, and your repos for projects 1, 2, and 3, and your weekly in class directories.  

However, I would suggest that you make a backup using one of the methods detailed in the following sections.

#### Backup option 1 - make a tarball, download to laptop/desktop, untar

First create a command line to your VM.

Issue a `pwd` command to make sure you are in the `/home/jupyter` directory.  If not, issue a `cd` to take your there and verify.

Create a tarball file using the following command. I like to put the date in the files so I know when I created the file.  You may want to update it with the current date.  I use the format YYYY_MM_DD so it sorts in a reasonable way:

```
tar cvfz mids_w205_backup_2020_04_25.tgz w205
```

```
zip -r mids_w205_backup_2020_04_25.zip w205
```

Verify the file is there using this command or similar:

```
ls -lh mids_w205_backup_2020_04_25.zip
```

In your command line window, in the upper right corner, there should be a keyboard symbol and a gear symbol.  The gear symbol has a dropdown menu.  Click the dropdown menu.  Choose the option "download file".  
