### GitHub branch issue: assignment branch is 1 commit ahead and 1 commit behind master

When you push your assignment branch up to GitHub, you may see the issue of the assignment branch is 1 commit ahead and 1 commit behind.

If you are editing files directly in the GitHub web interface, and also editing them in the clone on your VM, this will cause this issue.  When you edit a file in the assignment branch directly in GitHub, it will put it 1 commit ahead of your clone.  When you push from your clone, it will put it 1 commit ahead the branch in GitHub.

I've also seen bugs with GitHub classroom scripts that update branches which can also cause this issue.

Resolving the 1 commit ahead with 1 commit behind requires a really advanced knowledge of GitHub merging to complete.

However, an easier solution exists to solve this:  backup your work, delete the clone in your VM, delete the branch in GitHub, clone down the repo to the VM again, create the assignment branch, copy in your changes from your backups, state, commit, push.  

## Step 1: backup your work in your clone in the VM

The following commands will backup your entire w205 directory into a backup directory.  You will probably want to change the date to the current date.

```
cd

pwd
(make sure you are in /home/jupyter !!!)

mkdir w205_2021_05_25

cp -r w205/* w205_2021_05_25
```

Walk the backup directory structure and make sure everything you want to restore is backed up.


## Step 2: backup any changes in GitHub that are not present in the clone

If all of your changes are in the clone, you can skip this step.

If you have changes in GitHub that are not in the clone, view the file in raw mode in GitHub and copy the contents to a notepad on Widnows or TextMate on the Mac and save them into desktop files.

## Step 3: 

In GitHub, delete the assignment branch.

## Step 4:

In the VM, delete the clone (replace xxxxx with your GitHub username):

```
cd ~/w205

sudo rm -r project-1-xxxx
```
## Step 5:  In the VM, clone down the repo again, create the assigment branch:

```
cd ~/w205

git clone xxxxxxx

cd project-1-xxxxx

git branch assignment

git checkout assignment

git status
```

## Step 6: Replace files that you changed with the backup files

Selectively copy in files from the backup directory. Don't do a copy * as it will copy in git hidden files that will mess up the repo.  

If you had changes on the desktop, edit the files and copy in the content from the desktop.

## Step 7: Stage, commit, push

Stage, commit, and push from the clone to GitHub.


