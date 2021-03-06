# ibm-jenkins-slave-nvm

## Introduction

A jenkins slave image supports:

- [Docker in Docker](https://hub.docker.com/_/docker/)
- OpenJDK 8
- JNLP & SSH Slave
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

Currently published to [Docker hub: jackjiaibm/zowe-jenkins-slave-nvm](https://hub.docker.com/r/jackjiaibm/zowe-jenkins-slave-nvm/).

## Build And Test Image

To build image, run command `docker build -t jackjiaibm/zowe-jenkins-slave-nvm .`.

### Test Image With SSH Connection

To test run image, run command:

```
$ docker run --rm -it --privileged jackjiaibm/zowe-jenkins-slave-nvm <public key>

# or

$ docker run --rm -it --privileged -e "JENKINS_SLAVE_SSH_PUBKEY=<public key>" jackjiaibm/zowe-jenkins-slave-nvm
```

For example, use your local SSH public key:

```
$ docker run --rm -it --privileged jackjiaibm/zowe-jenkins-slave-nvm "$(cat ~/.ssh/id_rsa.pub)"
```

### Test Image With JNLP Connection

```
$ docker run --rm -it --privileged \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_SECRET=<jenkins secret>" \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_JAR_URL=<url to slave.jar>" \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_JNLP_URL=<url to jnlp>" \
  jackjiaibm/zowe-jenkins-slave-nvm
```

For example:

```
$ docker run --rm -it --privileged \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_SECRET=<jenkins secret>" \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_JAR_URL=https://wash.zowe.org:8443/jnlpJars/slave.jar" \
  -e "DOCKER_SWARM_PLUGIN_JENKINS_AGENT_JNLP_URL=https://wash.zowe.org:8443/computer/agent-st_pipeline__77-30269/slave-agent.jnlp" \
  jackjiaibm/zowe-jenkins-slave-nvm
```

After the Jenkins client container is started, you can use `docker ps` find the container and user `docker exec -it -u jenkins <container-id> bash` to connect to the container.

## About Base Image

`openjdk:8-jdk` which is Debian 9 (Stretch).

This docker image is a modified version from official [jenkins/jnlp-slave](https://hub.docker.com/r/jenkins/jnlp-slave) and [jenkins/ssh-slave](https://hub.docker.com/r/jenkins/ssh-slave). It can be used as `Connect method` - `Connect with SSH` with user `jenkins` if using `Docker Plugin`, or can be used as JNLP slave if using `Docker Swarm Plugin` (Kunernetes Plugin is not tested). The modification is installing nvm before declare "VOLUME /home/jenkins", so changes done by nvm install can be saved.

## About Node Version Manager

With nvm, the jenkins user doesn't require sudo to run `npm install`. This is explanation from npmjs.org: https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-one-reinstall-with-a-node-version-manager.
