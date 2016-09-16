---
layout: post
title: Dockerized build server

gist: a32c37b6bb548869db61a5ac0791e66c
---

Just like with the software we develop, we want to test the tools we use to
develop software. This post will show you how to have locally running build
stack in no-time.

## Docker compose

We're going to tie together to following services:

 * [Jenkins 2](https://hub.docker.com/_/jenkins/)
 * [Sonarqube](https://hub.docker.com/_/sonarqube/)
 * [PostgreSQL](https://hub.docker.com/_/postgres/)

After this it should be a piece of cake to extend this setup with Nexus or
GitLab.

Jenkins and Sonarqube both define alpine based versions. We'll be using these
since they're much smaller and should contain everything we need.

Also it appears that the ``./`` is required in front of the volume definitions
when using relative paths.

<script src="https://gist.github.com/remvos/{{ page.gist }}.js"></script>

As you can see the containers which are linked can use each others container
names as hostname. So later on, when configuring Sonar in Jenkins use
``http://sonar:9000/`` as Sonar location.

### Running the stack

```bash
cd `mktemp -d`
curl -O 'https://gist.githubusercontent.com/remvos/a32c37b6bb548869db61a5ac0791e66c/raw/7f236ca6b3838ea3437a4c078bf5e4c7cc3404b4/docker-compose.yml'
docker-compose up
```

Leave this terminal open for the logs.

## Set up Jenkins 2

Once all containers have started, Jenkins will show some message:

    jenkins_1   | *************************************************************
    jenkins_1   | *************************************************************
    jenkins_1   | *************************************************************
    jenkins_1   |
    jenkins_1   | Jenkins initial setup is required. An admin user has been created and a password generated.
    jenkins_1   | Please use the following password to proceed to installation:
    jenkins_1   |
    jenkins_1   | 43986230b7134ee9a58836d6ec85ce1e
    jenkins_1   |
    jenkins_1   | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
    jenkins_1   |
    jenkins_1   | *************************************************************
    jenkins_1   | *************************************************************

Copy the initial password and open [Jenkins](http://localhost:8080/). When
asked, simply select the default set of plugins. If you choose to be more
specific, that's fine too: your configuration and plugins are stored on you
host because all Jenkins data is mounted using a Docker volume.

Create your first admin user and start using Jenkins!

