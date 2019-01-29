### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Spring/synch_session_commands/checklist_b4_class_assignments.md

#### For assignment 4, we will be creating a branch in our docker container.  We will work only on the branch and leave the master branch untouched.  We will create a pull request on the branch.

First be sure you are in a docker container:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

Clone down the assignment 4 repo from inside the docker container.  Remember to use the method of going out to the GitHub website and using the green dropdown.  (If you have already cloned down assignment 4 and committed and pushed changes to the master branch that is ok.):
```
git clone link_to_your_repo_for_assignment_04
```

Change directory into your repo:
```
cd assignment-04-xxxx
```

Create a branch to work from.  (You can call the branch anything you want, however, if you call it assignment in all lower case, later in the semester, it will make it easiers to script against if you want to archive all of your assignment branches.):
```
git branch assignment
```

Switch to that branch:
```
git checkout assignment
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
git push origin assignment
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

