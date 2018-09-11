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

clone the course-content repo and the Assignment-00 repos:

```
cd w205

git clone linktorepo

(we will show you the best way to pull a link to a repo during class)

```




