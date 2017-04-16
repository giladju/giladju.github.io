---
layout: post
title:  "Learning new stuff with Docker, neo4j export/import"
date:   2017-02-06 11:14:49 +0200
categories: devops learn tips
---

## Introduction

Being in a consulting position, I often find myself being asked to perform tasks in technical areas I know close to nothing about.
For example I was asked to setup a Neo4J server in Amazon, and figure out how to export and import data from a developer's laptop with a local setup of the database.

I could spend days learning how Neo4J works, but instead I read up a bit on the official site: https://neo4j.com, and proceeded to run `neo4j` [on a docker container](https://neo4j.com/developer/docker)

If you are new to `docker` take the time to get familiar with [Docker images and containers](https://blog.docker.com/2016/05/docker-101-getting-to-know-docker/)

In this post I will detail the following steps:

* Preparing to run `neo4j` docker containers
* Running  a clean `neo4j` docker container
* In `neo4j` setting up a demo database
* Export the demo database
* Run a second clean `neo4j` docker container
* Import the dumped database 

## Running the `neo4j` docker

### Install Docker on Ubuntu

```
sudo apt-get -y install docker-engine
sudo usermod -aG docker $USER
sudo reboot
```

### Prepare your file system for two docker containers 

Prepare Logs and Data directories for first docker 

```
mkdir -p ~/neo4j/data ~/neo4j/logs
mkdir -p ~/neo4j/data2 ~/neo4j/logs2
```

### Docker pull `neo4j`

```
docker pull neo4j
```

### Make sure all docker containers are removed

```
docker rm `docker ps -a -q`
```

### Run vanialla `neo4j` docker

```
docker run \
--publish=7474:7474 --publish=7687:7687 \
--volume=$HOME/neo4j/data:/data \
--volume=$HOME/neo4j/logs:/logs \
neo4j
```
You should get:
```
Starting Neo4j.
2017-02-06 07:59:34.533+0000 INFO  No SSL certificate found, generating a self-signed certificate..
2017-02-06 07:59:35.686+0000 INFO  Starting...
2017-02-06 07:59:36.736+0000 INFO  Bolt enabled on 0.0.0.0:7687.
2017-02-06 07:59:41.491+0000 INFO  Started.
2017-02-06 07:59:43.586+0000 INFO  Remote interface available at http://localhost:7474/
```

Open [http://localhost:7474](http://localhost:7474/) in your browser

(Ignore the temporary red that might appear, you should be redirected to http://localhost:7474/browser)

![neo4j-initial-login]({{ site.url }}/assets/neo4j-initial-login.png)

<!---
{:class="img-responsive"})
-->

Login with password `neo4j` and set a new password

### Creating a demo database

At this stage there is no data on server, in order to create something, we will use the demo Movie/Actor database.

In the Editor type: `:play movie graph` like so:

![neo4j-editor-play-movie]({{ site.url }}/assets/neo4j-editor-play-movie.png)

And press Enter

Press the **Next** arrow on the right

Follow the instructions:

1. Click on the code block
2. Notice it gets copied to the editor above
3. Click the editor's play button to execute
4. Wait for the query to finish

The database is created and presented

![neo4j-demo-db-created]({{ site.url }}/assets/neo4j-demo-db-created.png)

### Export the demo database

In order to export the database you will need to stop the `neo4j` server, but stopping the `neo4j` server on a running docker will essentially stop the docker container completely. 

So we will do the following:

1. Stop the docker container - in the terminal from which you ran the container press `CTRL c`
2. Run a container with the same data, but this time run `/bin/bash` on it - allowing for interactive access to the server:

```
docker run \
--publish=7474:7474 --publish=7687:7687 \
--volume=$HOME/neo4j/data:/data \
--volume=$HOME/neo4j/logs:/logs \
-i -t neo4j /bin/bash
```

You should get a `bash` prompt:

```
bash-4.3# 
```

In it type:

```
./bin/neo4j-admin dump --database=graph.db --to=/data/2017-02-05.dump
```

Exit the docker

```
bash-4.3# exit
```

The docker container will stop

## Run a second clean `neo4j` docker container

Let's make sure there are no containers, running or stopped

```
docker rm -f `docker ps -a -q`
```

And let's double check we actually load a clean `neo4j` server

```
sudo rm -rf ~/neo4j/data2/*
```

Now, run the second container

```
docker run \
--publish=7474:7474 --publish=7687:7687 \
--volume=$HOME/neo4j/data2:/data \
--volume=$HOME/neo4j/logs2:/logs \
neo4j
```

Again - you should get

```
Starting Neo4j.
2017-02-06 10:30:18.970+0000 INFO  No SSL certificate found, generating a self-signed certificate..
2017-02-06 10:30:20.100+0000 INFO  Starting...
2017-02-06 10:30:20.917+0000 INFO  Bolt enabled on 0.0.0.0:7687.
2017-02-06 10:30:24.738+0000 INFO  Started.
2017-02-06 10:30:26.448+0000 INFO  Remote interface available at http://localhost:7474/
```

After logging in - you will note there is not data defined

![neo4j-no-data]({{ site.url }}/assets/neo4j-no-data.png)

Stop the docker container - in the terminal from which you ran the container press `CTRL c`

On your PC copy the `dump` file to the "volume" of the new docker container:

```
cp ~/neo4j/data/2017-02-05.dump ~/neo4j/data2/
```


Now run the a container with a `bash` prompt:

```
docker run \
--publish=7474:7474 --publish=7687:7687 \
--volume=$HOME/neo4j/data2:/data \
--volume=$HOME/neo4j/logs2:/logs \
-i -t neo4j /bin/bash
```

In the container prompt:

```
bash-4.3# bin/neo4j-admin load --from=/data/2017-02-05.dump --database=graph.db --force
```

Exit the docker

```
bash-4.3# exit
```

And start a container with `neo4j` running

```
docker run \
--publish=7474:7474 --publish=7687:7687 \
--volume=$HOME/neo4j/data2:/data \
--volume=$HOME/neo4j/logs2:/logs \
neo4j
```

Go to [http://localhost:7474/browser/](http://localhost:7474/browser/)

And note that there is a database in place:

![neo4j-with-data]({{ site.url }}/assets/neo4j-with-data.png)

Click the * Star to the right of `Movie` & `Person`

![neo4j-click-star]({{ site.url }}/assets/neo4j-click-star.png)

The graphic representation of the database should appear

![neo4j-graph-database]({{ site.url }}/assets/neo4j-graph-database.png)

# Summary

Learning something new can be a lot of a hard work, with the help of `docker` setting up a service is pretty quick, and in this case the specific task of learning how to export/import is now understood without the need to learn a whole lot of installation and configuration material.