### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Ownership issues between science and root

Files created in your droplet will be owned by science with group science. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **droplet** when logged in as science to change the owner to science and the group to science, recursively, for a directory:
```
sudo chown -R science:science w205
```

#### It's a good idea to always update the course-content repo prior to class
Note that if you made changes to your course-content repo, you won't be able to update it due to conflicts.  In that case, you will need to delete it and bring it down fresh.
```
cd ~/w205/course-content
git pull --all
cd
```

#### For assignment 3, we will be creating a branch in our docker container.  We will work only on the branch and leave the master branch untouched.  We will create a pull request on the branch.

First be sure you are in a docker container:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

Clone down the assignment 3 repo from inside the docker container.  Remember to use the method of going out to the GitHub website and using the green dropdown:
```
git clone link_to_your_repo
```

Change directory into your repo:
```
cd assignment-02-xxxx
```

Create a branch to work from:
```
git branch my_branch_name
```

Switch to that branch:
```
git checkout my_branch_name
```

Make changes to your README.md using vi:
```
vi README.md
```

Stage the changes:
```
git add README.md
```

Commit the changes:
```
git commit -m "meaningful comment goes here" 
```

Push the changes to GitHub:
```
git push origin my_branch_name
```

