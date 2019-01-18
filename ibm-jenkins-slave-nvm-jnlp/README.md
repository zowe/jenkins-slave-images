# ibm-jenkins-slave-nvm

## Introduction

A jenkins slave image supports:

- [Docker in Docker](https://hub.docker.com/_/docker/)
- OpenJDK 8
- SSH Slave
- [Node Version Manager](https://github.com/creationix/nvm)
- node.js v8.11.4 installed by nvm
- [JFrog CLI](https://jfrog.com/getcli/)
- Firefox v61.0.2 for headless Selenium test
- gnome-keyring and keytar
- Other Ubuntu packages:
  * curl
  * wget
  * pax
  * bzip2
  * rsync
  * vim
  * sshpass
  * jq

Currently published to [Docker hub: jackjiaibm/ibm-jenkins-slave-nvm](https://hub.docker.com/r/jackjiaibm/ibm-jenkins-slave-nvm/).

## Build And Test Image

To build image, run command `docker build -t jackjiaibm/ibm-jenkins-slave-nvm .`.

To test run image, run command:

```
$ docker run --rm -it --privileged jackjiaibm/ibm-jenkins-slave-nvm <public key>

# or

$ docker run --rm -it --privileged -e "JENKINS_SLAVE_SSH_PUBKEY=<public key>" jackjiaibm/ibm-jenkins-slave-nvm
```

For example, use your local SSH public key:

```
$ docker run --rm -it --privileged jackjiaibm/ibm-jenkins-slave-nvm "$(cat ~/.ssh/id_rsa.pub)"
```

After the Jenkins client container is started, you can use `docker ps` find the container and user `docker exec -it -u jenkins <container-id> bash` to connect to the container.

## About Base Image

`openjdk:8-jdk` which is Debian 9 (Stretch).

This docker image is a modified version from official `jenkins/ssh-slave`, and can be used as `Connect method` - `Connect with SSH` with user `jenkins`. The modification is installing nvm before declare "VOLUME /home/jenkins", so changes done by nvm install can be saved.

## About Node Version Manager

With nvm, the jenkins user doesn't require sudo to run `npm install`. This is explanation from npmjs.org: https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-one-reinstall-with-a-node-version-manager.
