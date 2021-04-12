## MIDS w205 - Backing up your work at the end of the semester

### GitHub classroom repos

GitHub classroom will delete all repos in the mids-w205-crook organization on GitHub.  This includes the repos for course content and projects 1, 2, and 3. 

If you forked a repo into the mids-w205-crook organization, the fork will also be deleted.

If you forked a repo in the mids-w205-crook organization into your own organization, I'm not sure if it will be deleted or not.  GitHub classroom does not have any documentation that seems to address this, and I don't have a way to test it.   You can fork a repo in the mids-w205-crook organization to your own organization, but I would recommend that you also make another backup of it.

### Your VM in the GCP

Your virtual machine (VM) in the Google Cloud Platform (GCP) will not go away, unless you delete it.  

The w205 directory should have all of your work for the semester, including course content, and your repos for projects 1, 2, and 3, and your weekly in class directories.  

However, I would suggest that you make a backup using one of the methods detailed in the following sections.

### Before doing anything, make a local backup in your VM for safe keeping !

Create a local backup in your VM using the following commands. I like to put the date in the files so I know when I created the file.  You may want to update it with the current date.  I use the format YYYY_MM_DD so it sorts in a reasonable way:

```
cd

pwd
(make sure you are in /home/jupyter !!!)

mkdir w205_2020_04_25

cp -r w205/* w205_2020_04_25

```

Verify the backup using the following command or similar:

```
ls -lhR w205_2020_04_25
```


### Backup option 1 - make a zipfile, download to laptop/desktop, unzip

This should work on either Windows or Mac.

First create a command line to your VM.

Create a zip file using the following commands. I like to put the date in the files so I know when I created the file.  You may want to update it with the current date.  I use the format YYYY_MM_DD so it sorts in a reasonable way:

```
cd

pwd
(make sure you are in /home/jupyter !!!)

zip -r mids_w205_backup_2020_04_25.zip w205

```

Verify the file is there using this command or similar:

```
ls -lh mids_w205_backup_2020_04_25.zip
```

In your command line window, in the upper right corner, there should be a keyboard symbol and a gear symbol.  The gear symbol has a dropdown menu.  Click the dropdown menu.  Choose the option "download file".   This may take a while.  Once it's done, it will be in the usual download folder that browser downloads are put in.

You can simply open and/or extract the zip file like you would any other zip file.

### Backup option 2 - make a tarball, download to laptop/desktop, untar

tarballs can be untarred on the Mac using a command line with the tar command.  

Windows cannot read tarballs using built in utilities.  You can download the open source utility 7zip which can untar, but it might just be easier to use option 1 with a zip file that Windows can read.

First create a command line to your VM.

Create a tarball file using the following commands. I like to put the date in the files so I know when I created the file.  You may want to update it with the current date.  I use the format YYYY_MM_DD so it sorts in a reasonable way:

```
cd

pwd
(make sure you are in /home/jupyter !!!)

tar cvfz mids_w205_backup_2020_04_25.tgz w205
```

Verify the file is there using this command or similar:

```
ls -lh mids_w205_backup_2020_04_25.tgz
```

In your command line window, in the upper right corner, there should be a keyboard symbol and a gear symbol.  The gear symbol has a dropdown menu.  Click the dropdown menu.  Choose the option "download file".   This may take a while.  Once it's done, it will be in the usual download folder that browser download are put in.

If you are using the Mac, you can open a command line and untar using the following command (be sure you are in the directory you want to untar it to:

```
tar xvf mids_w205_backup_2020_04_25.tgz
```

If you are using Windows with 7zip, you can open it with 7zip and extract from there.

### Cloning repos down to your desktop / laptop

You can also clone your GitHub repos down to your desktop / laptop.  It's probably easiest to just download the GitHub client for Windows or Mac and use that.

The downside to this is that you will only have the repos for course content and projects 1, 2, and 3.  

You won't have the directories for the weekly in class exercises that we did.

### Creating a single private repo in your private organization with all of the w205 directories in it

Another option would be to creat a single repo with all of your w205 content in it.  

This option is a bit more involved.

First, go into GitHub web interface and create a new private repo in your organization.   You will need to name it something other than w205, such as ucb_mids_w205, or you will need to rename the current w205 directory. Be sure you are creating it in your organization and not in mids-w205-crook, as mids-w205-crook repos will go away when GitHub Classroom resets for next semester.  Be sure you create a simple README.md as it's required.  Even a one liner is fine.  It just needs to have a README.md file.

Now to into your linux command line for your VM, cd into the home directory if you are not already there, and clone down the new private repo.  It will need to be a directory parallel to the existing w205 and not under w205 otherwise it will mess up all of your repos.  Please double check that you are in the home directory before doing this.:

```
cd

pwd
(make sure you are in /home/jupyter !!!)

git clone xxx
(where xxx is a https link to your repo from the green dropdown)

cp -r w205/* xxx
(where xxx is the directory for your new repo such as ucb_mids_w205)

cd xxx
(where xxx is the directory for your new repo)

```

Repos nested in side of other repos is not allowed.  GitHub will choke on this, so we need to remove the .git subdirectories.

The following command will find and remove all .git directories.  Be very careful.  Make sure you did a backup before attempting this!

```

pwd
(make sure you are in your new repo!)

find . -name .git | grep -v "\./.git" | xargs rm -rf

```

Add everything, commit, push to GitHub.

```
git add *

git commit -m "adding everything from w205"

git push origin main

```

Go to the GitHub web interface and verify that the new repo has everything.  After you have verified, you will now have a repo with everything from w205 in it.


