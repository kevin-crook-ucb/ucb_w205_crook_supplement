### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md

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

#### Create a pull request.  This is done in the GitHub web gui.  Pull Requests are part of GitHub, but not part of git command line.  Follow the instructions on the slide.

#### Docker 

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


