# Getting started with your Digital Ocean (DO) Droplet

This document will cover the basics of how to connect and login to your digital ocean droplet, how to run git on the command line to interface with GitHub, and how to run Docker in your droplet.

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

Find your home directory using one of the following commands:
```
echo $HOME
```
```
cd 
pwd
```
```
cd ~
pwd
```

Using a Chrome web browser on your desktop, 
  
  
