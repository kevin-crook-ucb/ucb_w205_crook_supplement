### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Fall/synch_session_commands/checklist_b4_class_assignments.md

#### Docker

Run the regular container
```
docker run -it --rm -v ~/w205:/w205 midsw205/base:latest bash
```

What containers are running right now?
```
docker ps
```

What containers exist?
```
docker ps -a
```

What images do I have?
```
docker images
```

Clean up containers
```
docker rm -f name_of_container
```

Idiomatic docker
```
docker run -it --rm -v ~/w205:/w205 midsw205/base pwd
```

#### Branches and Pull Requests in GitHub (revisited)

Clone a repo from GitHub
```
cd ~/w205
git clone <link_to_repo>
```

Change directory into your repo:
```
cd <repo directory>
```

Check to see what branch we are on:
```
git status
```

You should be on the assignment branch.  If not, then you will need to create a branch called assignment and switch to that branch. 

Create a branch called assignment:
```
git branch assignment
```

Switch to that branch:
```
git checkout assignment
```

Verify that you are on the assignment branch.  Note that if you never move off the assignment branch, this clone will remain on the assignment branch:
```
git status
```

As needed: make edits to existing files, create new files.  You will see edited or new files tracked with a status check.

Stage the changes.  Note that you should only stage the file that you want to be in the repo.  Checksum files, swap files, data files, etc. you may not want to put into your repo:
```
git add README.md
```

Commit the changes:
```
git commit -m "meaningful comment goes here" 
```

Push the changes to GitHub:
```
git push origin assignment
```

You may edit files, add new files, stage, commit, and push as often as you wish.  

Once you are done and want to turn in the assignment, you should create a pull request comparing assignment branch to master branch.

You should have only 1 pull request, and not make any changes after the pull request. 

What if you want to make a change after the pull request?  Then close the pull request without merge, make changes, stage, commit, and push.  Once you are done, create a new pull request.
