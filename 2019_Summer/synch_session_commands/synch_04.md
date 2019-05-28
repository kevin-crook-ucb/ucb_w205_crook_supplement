### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Summer/synch_session_commands/checklist_b4_class_assignments.md

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

Clone a repo from GitHub
```
cd ~/w205
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

Make changes to code
(use nano or vi)
```
nano README.md

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
