# Getting started with your Digital Ocean (DO) Droplet

This document will cover the basics of how to connect and login to your digital ocean droplet, how to run git on the command line to pull down the course content repo, and how to run Docker in your droplet.

## Connecting to and logging into your droplet

Your instructor will give you an IP address and a password via slack

Username is science

Secure Shell (ssh) will be used, which requires port 22 to be unblocked for outbound connections.  If port 22 is blocked you will need to open the port on your router, and if that is not possible, you will need to use another network.

### Windows Users:

[Download PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/)

It is recommended that you download the portable version rather than the installed version, the zip file containing everything usually works best, but be sure to unzip after you download it or it will not work correctly.

Run PuTTY, enter your IP address for the Hostname.  If you want you can set the font you want to use on the left menu with Window => Appearance.  If you want, you can save the setting using the Save button and load saved settings using the Load button.  To open a session, click the Open button.  Enter science for the username and enter the password given to you by your instructor.

Once you have a terminal open, you can easily open more by clicking the upper left and selecting New Session.  You can have as many sessions open as you like and it's generally recommended to have several open.
 
### Mac Users:

Open a command line and use the ssh command as follows:  

```
ssh science@your_ip_address
```

Where your_ip_address is replaced by the one given to you by your instructor.

When prompted for password, enter the password given to you by your instructor.

You can repeat the process in other command lines to have multiple sessions open as you like and it's generally recommended to have several open.

## Run git on the command line and connect to GitHub

Find your username using the following command:
```
whoami
```

Find your home directory using the following commands:
```
echo $HOME
```

Issue the following command to clone down the course content repo:
```
cd
git clone https://github.com/mids-w205-crook/course-content
```

You can look at an copy files from this repo as much as you want, but if you make changes to the repo, you may not be able to synch with the GitHub repo to pull updates in the future.

Issue the following command to update your course content repo.  The course content repo is being updated with fixes all the time, so it' recommended that before each class and before you work on anything requiring the repo, that you pull the latest using the following command:

```
cd ~/course-content
git pull --all
```

## Run Docker in your droplet

Move to your home directory:
```
cd
```

Check your Docker version:
```
docker --version
```

Check your Docker Compose version:
```
docker-compose --version
```

Check your Docker info:
```
docker info
```

Run Docker hello world test program:
```
docker run --rm hello-world
```

Run the mids w205 image with a bash shell connected to a terminal and local volume mounted:
```
cd
mkdir w205
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

## Docker cleanup - you may want to run a quick cleanup of all containers and images before class and before 

See the Docker containers that are currently running:
```
docker ps
```

See the Docker containers whether running or not:
```
docker ps -a
```

Remove a Docker container (not running):
```
docker rm my_container
```

Remove a Docker container (even if running):
```
docker rm -f my_container
```

Remove all Docker containers that are not running:
```
docker rm $(docker ps -aq)
```

Forceably remove all Docker containers, including those that are currently running:
```
docker rm -f $(docker ps -aq)
```

See the Docker images:
```
docker images
```

Remove a Docker image:
```
docker rmi my_image_name
```

Remove all Docker images:
```
docker rmi $(docker images -aq)
```

