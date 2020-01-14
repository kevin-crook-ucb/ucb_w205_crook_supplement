### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #1

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2020_Spring/checklist_b4_class_assignments.md

#### Checklist for Today's Class

- Introductions (we will wait until we get everyone going on GCP setup)

- Everyone join the slack channels **w205** and **w205-crook-2020-spring**

- Everyone ping me on slack using the following template:
  * W205 Spring 2020
  * Last Name:
  * First Name:
  
- Google Form - everyone fill out

- GitHub repo https://github.com/mids-w205-crook/course-content 
  * Everyone verify access AFTER I add you based on Google Form
  * Everyone clone down to their Virtual Machine

- Google Cloud Platform (GCP) - everyone sign up, enable billing, first project, enable APIs

- GCP => AI Platform => Notebooks => Open Jupyterlab
  * Everyone verify they can open a Jupyter Notebook
  * Everyone verify they can open a linux command line using Jupyter: Compute Engine => VM Instances => SSH
  * Everyone verify they can open a linux command line using SSH on the Virtual Machine: Settings Gear Upper Right => Change Linux Login => jupyter
  
- Docker - veryify everyone can run
  
- GitHub Classroom
  * Everyone accept the link for the signup assignment, find their name, wait for signup assignment repo to be created (may take several minutes)
  * Clone down the signup assignment, create branch, track, stage, commit, push, create pull request.
  
- Syllabus 

#### Updating Docker and running Docker in your Virtual Machine

```
docker pull midsw205/base

mkdir ~/w205

docker run -it --rm -v ~/w205:/w205 midsw205/base:latest bash

(use exit to exit the container)
```

#### Using git command line

clone the course-content repo and the GitHub Classroom assignment repos:

```
cd ~/w205

git clone <link_to_repo>
```

The best way to get a link to a repo is to go into the repo on GitHub web interface and use the green dropdown "Clone or download" button.

#### signup assignment (not graded - just for practice)

in the previous commands, we cloned, the signup repo to our droplet.

change directories into the repo directory

edit the README.md file using either vi (preferred) or nano (easier alternative to vi, although not always available)

```
cd ~/w205

git clone <link_to_repo>

cd <repo directory>

git status

git branch assignment

git status

git checkout assignment

git status

<edit the README.md file using vi or nano and save your changes>

git status

git add README.md

git status

git commit -m "my new readme"
```

the first time you commit, it doesn't know who you are.  use the following command to fix.  replace the xxx's with your email address and your github username:

```
git config --global user.email "xxxxx"

git config --global user.name "xxxx"
```

after setting these continue again with the commit:

```
git commit -m "my new readme"

git status

git push origin assignment

git status

go to the github web interface for your repo, refresh if needed, verify your changes made it.  You may also make a pull request there and only select your instructor as a reviewer.
```
