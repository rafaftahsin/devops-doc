---
title: Class 2
parent: 'Module 8'
layout: page
nav_order: 2
---

আজকের ক্লাসের কাজ হচ্ছে আমাদের backend application টাকে আমরা dockerize করব ... 

docker is just a virtualize environment ... right ? same as before we will start with ubuntu as our base image ... 

so let's start ... 

### Step 1 : Start with `Ubuntu` base image

<br>

Run the following command. Make sure you ran the command from your repository base location.

```
# docker run -it -v $(pwd):/app -p 8080:8080 ubuntu
docker run -it -v $(pwd):/app --network=host ubuntu
```

Ok. After we run above command, our terminal will be attached to container's terminal. We have our repository linked at `/app` location inside the container and we also have exposed container's 8080 port and mapped the port to host's 8080 port.

### Step 2 : Install dependencies

<br>

Update ubuntu's apt repository and install all the dependencies that we need to run our backend 

```
root@a21d865e2aa3:/# apt update && apt install -y openjdk-17-jdk 
```

Confirm Java 17 is installed

```
root@a21d865e2aa3:/# java --version 
openjdk 17.0.16 2025-07-15
OpenJDK Runtime Environment (build 17.0.16+8-Ubuntu-0ubuntu124.04.1)
OpenJDK 64-Bit Server VM (build 17.0.16+8-Ubuntu-0ubuntu124.04.1, mixed mode, sharing)
```

openjdk-17-jdk also installs the jre. So we don't need to think about the runtime. Now we need to install another dependency `maven`

```
root@a21d865e2aa3:/# apt install -y maven
```

With installing `maven`, all our dependencies are now installed.

### Step 3 : Building the application

```
root@a21d865e2aa3:/# cd /app/
root@a21d865e2aa3:/app# mvn package -Dmaven.test.skip
```

As we might have some failed test cases, we have skipped the test cases with `-Dmaven.test.skip`


We now have a runnable jar file 

```
root@ip-172-31-9-208:/app# ls -al target/devops-demo-0.0.1-SNAPSHOT.jar
-rw-r--r-- 1 root root 63072155 Oct 12 14:49 target/devops-demo-0.0.1-SNAPSHOT.jar
```

### Step 4: Run the application


```
java -jar target/devops-demo-0.0.1-SNAPSHOT.jar
```

---

# Automating Steps with `Dockerfile`

### Writing `Dockerfile`

Let's recap what we did manually to run our application inside a container. 

Step 1: We started with a base iamge Ubuntu 
Step 2: We have installed dependencies
Step 3: We built our application
Step 4: We ran our application

So, with `Dockerfile` we will automate the steps. Let's open a file named `Dockerfile` in our repository

```Dockerfile
# Step 1: We are defining our base image
FROM ubuntu:22.04                                          

# Copying our application in /app directory
COPY ./ /app

# Fix our working directory to our app directory where we copied our source code
WORKDIR /app                                              

# Tells docker not to ask tzdata and other interactive questions
ARG DEBIAN_FRONTEND=noninteractive 

# Step 2: Installing dependencies
RUN apt update && apt -y install openjdk-17-jdk maven     

# Step 3: Building Application
RUN mvn package -Dmaven.test.skip

# Step 4: Run the application
ENTRYPOINT ["java", "-jar", "target/devops-demo-0.0.1-SNAPSHOT.jar"]
```

### Building image from `Dockerfile`

Our `Dockerfile` is ready. Now it's time to build the image from `Dockerfile`

```
docker build -t rafaftahsin/devops-demo-backend:v4 .
```

### Run application

With docker image being built, let's run it 

```
docker run --network=host rafaftahsin/devops-demo-backend:v4
```

# docker run with env file 

# change host network 


# docker compose 

# Multi stage docker build 