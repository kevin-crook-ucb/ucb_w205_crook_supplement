# UCB MIDS W205 Summer 2018 - Kevin Crook's agenda for Synchronous Session #1

## Introductions

## Review the syllabus and the instructor's syllabus supplement

## Ensure that everyone has gone through the "Checklist Before Your First Synchronous Class Meeting"

## Connect to your droplet

Your instructor will send you via slack your droplet ip address, username, and password.

Windows Users will need to use a terminal emulator such as PuTTY [https://www.chiark.greenend.org.uk/~sgtatham/putty/]. There are two options, 1) an install or 2) downloading a zip file, unzipping, and running in place.  Option #2 is recommended.  Please be sure you unzip before trying to run it.

Mac Users will use the terminal with ssh
```
ssh username@<ip address>
```

**Please be sure you manually type in both the username and password keystroke by keystroke (do not paste, it will not work!)**

**Breakout:** ensure that everyone is connected to their droplet

##  Run docker with the mids image in your droplet and test the directory mount

```
docker pull midsw205/base
```

```
mkdir w205
```

```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

```
df -k
```

```
vi myfile.txt
```

**Breakout:** save the file.  open another terminal window.  verify that myfile.txt show up both in docker and in the droplet.  exit docker and verify the file outlives the container.  create a new docker container and verify it is present.

```
exit or control-d to exit the container
```

## Using git command line inside the container, clone the course content repo, go over how to update it (be sure and do this inside the docker container!)

```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

Review how to properly copy a repo link using GitHub. 
```
git clone <repo>
```

The course content repo gets frequently updated.  It's a best practice to always update the course-content repo.  Also, do not make any changes to this directory, as you will not be able to update it.
```
cd ~/course-content
```
```
git pull --all
```

**Breakout:** ensure that everyone has properly cloned their repo inside the docker container and can update it

```
exit or control-d to exit the container
```

## Walk through assignment-00

```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

Retrive you repo for assignment-00 using the technique previously demonstrated and clone it: 
```
git clone <repo>
```

```
cd <repo>
```

```
git status
```

```
git branch assignment
```

```
git status
```

```
git checkout assignment
```

```
git status
```

```
vi README.md
```

```
git status
```

```
git add README.md
```

```
git status
```

```
git commit -m "change to README.md"
```

First time you commit to a repo, it asks you to set your email and username, if you have not done so already:
```
git config --global user.email "you@email.com"
git config --global user.name "Your Name"
```

```
git commit -m "change to README.md"
```

```
git status
```

```
git push origin assignment
```

```
git status
```

Using GitHub, verify the main branch is untouched, verify the assignment branch exists, verify no other branches exist, verify committs only to the assignment branch.

**Breakout:** ensure that everyone has done the above

Exit the container, create a new container of the same, make another simple change to the branch and commit it.

Using GitHub, create 1 and only 1 pull request to your instructor on the last commit of the assignment branch.

**Breakout:** ensure that everyone has followed the above instructions.  Verify each other's repos.


## assignment-01 getting started

For assignment-01, let's create a docker container, clone the repo, create an assignment branch, checkout the branch, make a simple change to the README.md, stage the change, commit the change to the branch, push the branch to GitHub.  Do not create a pull request until your last commit and you are ready to submit.

**Breakout:** ensure that everyone has followed the above instruction. Verify each other's repos.
