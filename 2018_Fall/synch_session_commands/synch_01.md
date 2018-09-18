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

git clone linktorepo

The best way to get a link to a repo is to go into the repo on GitHub web interface and use the green dropdown "Clone or download" button.

```

#### assignment-00 (formerly known as the sign up assignment)

in the previous commands, we cloned, the assignment-00 repo to our droplet.

change directories into the repo directory

edit the README.md file using either vi (preferred) or nano (easier alternative to vi, although not always available)

```

git status

git add README.md

git commit -m "my new readme"

the first time you commit, it doesn't know who you are.  use the following command to fix.  replace the xxx's with your email address and your github username:

git config --global user.email "xxxxx"

git config --global user.name "xxxx"

git commit -m "my new readme"

git push origin master

go to the github web interface for your repo, refresh if needed, verify your changes made it.

```



