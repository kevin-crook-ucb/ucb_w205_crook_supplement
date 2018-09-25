### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #5

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Ownership issues between science and root

Files created in your droplet will be owned by science with group science. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **droplet** when logged in as science to change the owner to science and the group to science, recursively, for a directory:
```
sudo chown -R science:science w205
```

#### It's a good idea to always update the course-content repo prior to class
Note that if you made changes to your course-content repo, you won't be able to update it due to conflicts.  In that case, you will need to delete it and bring it down fresh.
```
cd ~/w205/course-content
git pull --all
cd
```

#### Redis

```
docker run redis
(control c to exit)

docker run -d redis
(control c to exit)

docker run -d --name redis redis
(control c to exit)

docker run -d --name redis -p 6379:6379 redis
(control c to exit)
```


