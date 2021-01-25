### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #4

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Checklist before class and before working on assignments

Right now, this checklist has things in it we haven't covered yet, so just do what we have covered.

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2021_Spring/checklist_b4_class_assignments.md


#### Docker

Run the regular container
```
docker run -it --rm -v ~/w205:/w205 midsw205/base:latest bash
```

What containers are running right now?
```
docker ps
```

What containers exist?
```
docker ps -a
```

What images do I have?
```
docker images
```

Clean up containers
```
docker rm -f name_of_container
```

Idiomatic docker
```
docker run -it --rm -v ~/w205:/w205 midsw205/base pwd
```

#### Project 1 

We will use the balance of time to discuss project 1.

We have covered GitHub procedures already, so we will only revisit these procedures in class if student would rather do this than spend time discussing the project.
