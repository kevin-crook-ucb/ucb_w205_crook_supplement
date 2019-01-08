### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #1

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2018_Fall/synch_session_commands/checklist_b4_class_assignments.md

#### Mac users - connect to droplet

```
ssh science@ipaddress
```
password does not echo!

#### Windows users - use PuTTY

https://www.putty.org/
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

Recommended download: Alternative Binary Files => putty.zip => 64 bit
make a directory, download the zip file, extract it, we will use putty.exe and puttygen.exe

#### Updating Docker and running Docker in your droplet

```
docker pull midsw205/base

mkdir w205

docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash

(use exit to exit the container)
```

#### Using git command line

clone the course-content repo and the assignment-00 repos:

```
cd w205

git clone link_to_repo
```

The best way to get a link to a repo is to go into the repo on GitHub web interface and use the green dropdown "Clone or download" button.

#### assignment-00 (formerly known as the sign up assignment)

in the previous commands, we cloned, the assignment-00 repo to our droplet.

change directories into the repo directory

edit the README.md file using either vi (preferred) or nano (easier alternative to vi, although not always available)

```
git status

git add README.md

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

git push origin master

go to the github web interface for your repo, refresh if needed, verify your changes made it.
```
