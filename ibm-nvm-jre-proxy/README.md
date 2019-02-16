# ibm-nvm-jre-proxy

## Introduction

A jenkins slave image supports:

- OpenJDK 8 JRE
- node.js v6.14.4 installed by nvm
- Other Alpine packages:
  * curl

Currently published to [Docker hub: jackjiaibm/ibm-nvm-jre-proxy](https://hub.docker.com/r/jackjiaibm/ibm-nvm-jre-proxy/).

## Build And Test Image

To build image, run command `docker build -t jackjiaibm/ibm-nvm-jre-proxy .`.

### Test Image

To test run image, run command:

```
$ docker run --rm -it -v $PWD/app-folder:/app -p 7554:80 jackjiaibm/ibm-nvm-jre-proxy
```

The `./app-folder` can have these content:

- **nginx-conf.d**: nginx configurations. The config should define a server listen on port 80.
- **.env**: environment variables in KEY=VALUE format for each line.
- **boorstrap.sh**: application bootstrap script to start the application.
