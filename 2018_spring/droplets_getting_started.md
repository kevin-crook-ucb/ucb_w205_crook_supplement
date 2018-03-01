# Getting started with your Digital Ocean (DO) Droplet

This document will cover the basics of how to connect and login to your digital ocean droplet, how to run Docker in your droplet, how to run git on the command line to pull down the course content repo, how to cleanup Docker, and suggestions to prepare for class and for working on your assignments.

## Connecting to and logging into your droplet

Your instructor will give you an IP address and a password via slack

Username is science

Secure Shell (ssh) will be used, which requires port 22 to be unblocked for outbound connections.  If port 22 is blocked you will need to open the port on your router, and if that is not possible, you will need to use another network.

### Windows Users:

Down PuTTY (https://www.chiark.greenend.org.uk/~sgtatham/putty/)

It is recommended that you download the portable version rather than the installed version, the zip file containing everything usually works best, but be sure to unzip after you download it or it will not work correctly.

Run PuTTY, enter your IP address for the Hostname.  If you want you can set the font you want to use on the left menu with Window => Appearance.  If you want, you can save the setting using the Save button and load saved settings using the Load button.  To open a session, click the Open button.  Enter science for the username and enter the password given to you by your instructor.

Once you have a terminal open, you can easily open more by clicking the upper left and selecting Duplicate Session.  You can have as many sessions open as you like and it's generally recommended to have several open.
 
### Mac Users:

Open a command line and use the ssh command as follows:  

```
ssh science@your_ip_address
```

Where your_ip_address is replaced by the one given to you by your instructor.

When prompted for password, enter the password given to you by your instructor.

You can repeat the process in other command lines to have multiple sessions open as you like and it's generally recommended to have several open.

Mac users may want to consider a third party terminal emulation software if you want more functionality than the command line provides.  PuTTY is not available for the Mac.

## Security warning message

The first time you login to a particular host using ssh, you will get a security warning message in a popup.  Anytime you use Secure Sockets Layer (SSL), which is used by Secure Shell (ssh), it will attempt a third party authentication.  For example, if you go to a website such as google.com using the secure version of http called https, it will attempt third party authentication with a certificate authority.  If the certficate is registered and deemed to be authentic, the public key from the server will automatically be downloaded.  If the certificate is not registered or deemed invalid or expired, you will get a warning message.  Since our droplets are not registered, we will get this warning message. Go ahead and accept the public key.  Note that anytime you run virtual machines in any cloud service you will get this warning.

## Inside Docker / Outside Docker

Some commands will be run inside of a Docker container.  Some commands will be run outside a Docker container.

One rule of thumb:  if the command starts with "docker" or "docker-compose" then it's to be run outside of a docker container.

How to tell if you are inside Docker or outside Docker: at the left of the command prompt is the username.  If the username is science then you are not in Docker, you are at the droplet command line.  If the username is root and the hostname is a long hex string, then you are in a container.

## Find your droplet home directory and make the w205 directory

Find your home directory using the following commands:
```
echo $HOME
```

Create the w205 directory:
```
cd
mkdir w205
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

## Run the mids w205 image with a bash shell connected to a terminal and local volume mounted:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

## Run git on the command line and connect to GitHub

Issue the following command to clone down the course content repo (inside container):
```
cd
git clone https://github.com/mids-w205-crook/course-content
```

You can look at an copy files from this repo as much as you want, but if you make changes to the repo, you may not be able to synch with the GitHub repo to pull updates in the future.

Issue the following command to update your course content repo.  The course content repo is being updated with fixes all the time, so it' recommended that before each class and before you work on anything requiring the repo, that you pull the latest using the following command (inside container):

```
cd ~/course-content
git pull --all
```

## Docker cleanup commands

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

# Suggestion - before class and before you do any substantial work

* Always pull down the latest version of course content GitHub repo as there are frequent fixes.  You don't want to be chasing a bug that's already been fixed
* Always check for an remove old containers, unless you specifically know what they are and are attempting to restart them.
* If you are getting strange errors, it may be image corruption.  Remove all images and bring them down again.  Since it's in a droplet in the cloud, it has very high speed Internet connections and will be pretty fast. 

