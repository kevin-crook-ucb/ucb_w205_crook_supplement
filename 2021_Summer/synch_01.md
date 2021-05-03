### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #1

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist for Today's Class

- Introductions
  * Name
  * Preferred name you would like to be called
  * Preferred pronouns
  * Where you are located
  * Education background
  * Work background
  * Experience with Data Science
  * Experience with Data Engineering

- Everyone join the slack channels **w205** and **w205-crook-2021-summer**
  * w205 channel is all sections, all instructions, all TAs.  
  * technical issues should go to w205 so TAs can pick them up and everyone can benefit
  * w205-crook-2021-summer should be non-technical issues related to my sections

- Everyone send me a DM (direct message) on slack using the following template:
  * W205 2021 Summer
  * Last Name:
  * First Name:
  
- Google Form - everyone fill out
  * Link will be posted to slack so it isn't publicly available

- GitHub repo https://github.com/mids-w205-crook/course-content 
  * Everyone verify access after I add you based on Google Form
  * It may take me a while to add you, it won't be instantly

- Syllabus 
  * README.md file of the course-content repo
  * https://github.com/mids-w205-crook/course-content/blob/master/README.md

- Readings
  * Most are available from the Berkeley library at no cost.
  * You have to be setup with the Berkeley library. 
  * The library has support if you are having difficulty finding the books.
  * If you cannot find a book, please post to the w205 channel.

- mids-w205-crook GitHub organization
  * Everyone verify they can see and change to the organization
  * Practice changing organizations
  * Please do not fork or create additional repos to this organization, as it's very confusing to other students

- GitHub Classroom
  * Links will be posted to slack so they aren't publicly available
  * Be sure you are logged into the correct GitHub account before clicking a link
  * Choose your name from the list.  Please don't skip this.  Let me know if you name isn't there.
  * Please wait until one repo is created and verified before starting the next repo or you will risk corrupting them.

- Google Cloud Platform (GCP)
  * Chrome browser
  * First time individual, non-business users get $300 credit - if you have used GCP before you, even a free trail or an education credit, you may not get the $300
  * Individual not business
  * Must give them a credit card to get the credit.  If you don't give a credit card, you may loose the $300 credit.
  * Be sure you are logged into the right Google account (most of us have several Google accounts)
  * Using an education email, such as berkeley.edu, will increase your chances of getting education credits in the future

- Create a VM (virtual machine) on the GCP 
  * GCP => AI Platform => Notebooks 
  * May need to enable the notebooks API (GCP makes frequent changes, so I don't have the current sequence for this)
  * Near the top, to the right of the word "Notebooks" => NEW INSTANCE
  * Choose Python 3 in the dropdown
  * Region: us-west1 (Oregon) Zone: us-west1-b (seems to have the best luck, quite a few regions have technical issues)
  * Machine type: 4 vCPUs, 15 GB RAM
  * Book disk: 100 GB Standard persistence disk
  * Data disk: 100 GB Standard persistence disk
  
- VM
  * How to start
  * How to stop
  * How to create a JupterLab window
  * How to create an SSH Linux Command Line
  * How to verify the SSH Linux Command Line is logged in as jupyter and change it if it's not
  
- Docker - veryify everyone can run
  
- Clone GitHub repos to the VM in GCP
  * course-content
  * signup assignment - walk though creating branch, making a change to a branch, tracking changes, staging changes, committing changes, pushing changes, and creating a pull request
  * project 1
  * project 2
  * project 3
  
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
