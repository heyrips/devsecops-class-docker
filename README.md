# devsecops-class-docker

## Pre requisiteis

- launch ubuntu ec2
- Perform the following steps in ubuntu ec2

## Update OS

```
sudo apt-get update -y
sudo apt-get upgrade -y
```

## Install Docker

```
sudo wget https://download.docker.com/linux/ubuntu/gpg
sudo apt-key add gpg
```

```
sudo vi /etc/apt/sources.list.d/docker.list
```

add this to this file

- deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable'

```
sudo apt-get update -y
sudo apt-get install docker-ce -y

sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
```

## Create Docker Volume for Data and Log

```
sudo docker volume create jenkins-data
sudo docker volume create jenkins-log
sudo docker volume ls
```

## Install Jenkins with Docker

1. create docker folder
2. Create Dockerfile

```
FROM jenkins/jenkins
LABEL maintainer="hitjethva@gmail.com"
USER root
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins
USER jenkins

ENV JAVA_OPTS="-Xmx8192m"
ENV JENKINS_OPTS="--logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war"
```

## Build the docker image

```
cd docker
docker build -t devops-jenkins:1.0 .
```

## Run Jenkins Container with Data and Log Volume

```
sudo docker run -p 8080:8080 -p 50000:50000 \
    --name=jenkins-master \
    --mount source=jenkins-log,target=/var/log/jenkins \
    --mount source=jenkins-data,target=/var/jenkins_home \
    -d devops-jenkins:1.0

sudo docker ps
```

## Login to the container

```
sudo docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
```

## Access jenkins

http://public-ip:8080

- Reference: https://container-devsecops.awssecworkshops.com

### Dockerfile linitng

- Hadolint
- https://github.com/hadolint/hadolint
- sudo docker run --rm -i -v ${PWD}/hadolint.yml:/.hadolint.yaml hadolint/hadolint:v1.16.2 hadolint -f json - < Dockerfile

### Secrets scanning

- trufflehog
- https://github.com/trufflesecurity/trufflehog
- sudo docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/trufflesecurity/test_keys

## Vulnerabity scanning

- Anchore
- https://docs.anchore.com/current/docs/using/cli_usage/images/
- sudo curl -sSfL https://anchorectl-releases.anchore.io/anchorectl/install.sh | sudo sh -s -- -b /usr/local/bin

### Push to DockerHub

- docker login
- sudo docker tag devops-jenkins partha2019/devops-jenkins:1.0
- sudo docker push partha2019/devops-jenkins:1.0

### Download to another Instance

- launch the amazon linux2 instance
- install docker
- get the image
  - docker pull partha2019/devops-jenkins:1.0
- run the image

sudo docker run -p 8080:8080 -p 50000:50000 \
 --name=devops-jenkins \
 -d devops-jenkins:1.0
