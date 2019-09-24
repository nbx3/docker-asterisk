# The `mlan/asterisk` repository

![travis-ci test](https://img.shields.io/travis/mlan/docker-asterisk.svg?label=build&style=popout-square&logo=travis)
![image size](https://img.shields.io/microbadger/image-size/mlan/asterisk.svg?label=size&style=popout-square&logo=docker)
![docker stars](https://img.shields.io/docker/stars/mlan/asterisk.svg?label=stars&style=popout-square&logo=docker)
![docker pulls](https://img.shields.io/docker/pulls/mlan/asterisk.svg?label=pulls&style=popout-square&logo=docker)

THIS DOCUMENT IS UNDER DEVELOPMENT AND CONTAIN ERRORS

This (non official) repository provides dockerized PBX.

## Features

Feature list follows below

- Asterisk PBX
- php webhook for (incoming) SMS http ISTP origination
- dialplan curl (outgoing) SMS http ISTP termination
- dialplan ISTP originating (incoming) SIP voice call
- dialplan ISTP termination (outgoing) SIP voice call
- Alpine Linux

## Tags

The breaking.feature.fix [semantic versioning](https://semver.org/)
used. In addition to the three number version number you can use two or
one number versions numbers, which refers to the latest version of the 
sub series. The tag `latest` references the build based on the latest commit to the repository.

The `mlan/asterik` repository contains a multi staged built. You select which build using the appropriate tag from `mini`, `base`, `full` and `xtra`. The image `mini` only contain Asterisk.
To exemplify the usage of the tags, lets assume that the latest version is `1.0.0`. In this case `latest`, `1.0.0`, `1.0`, `1`, `full`, `full-1.0.0`, `full-1.0` and `full-1` all identify the same image.

# Usage

Often you want to configure Asterisk and its components. There are different methods available to achieve this. Moreover docker volumes or host directories with desired configuration files can be mounted in the container. And finally you can `docker exec` into a running container and modify configuration files directly.

If you want to test the image right away, probably the best way is to use the `Makefile` that comes with this repository.

To build, and then start a test container you simply have to `cd` into the repository directory and type

```bash
make build test-up
```

The you can connect to the asterisk command line interface (CLI) running inside the container by typing

```bash
make test-cli
```

From the Asterisk CLI you can type

```bash
pjsip show endpoints
```

to see the endpoints (soft phones) that are configured in the `/etc/asterisk/pjsip_wizard.conf` configuration file that comes with the image by default.

When you are done testing you can destroy the test container by typing

```bash
make test-down
```

## Docker compose example

An example of how to configure an web mail server using docker compose is given below. It defines 4 services, `pbx-app`, `pbx-mta`, `pbx-db` and `auth`, which are the web mail server, the mail transfer agent, the SQL database and LDAP authentication respectively.

```yaml
version: '3.7'

services:
  tele:
    image: mlan/asterisk
    restart: unless-stopped
    cap_add:
      - sys_ptrace
    networks:
      - proxy
    ports:
      - "80:80"
      - "5060:5060/udp"
      - "10000-10009:10000-10009/udp"
    volumes:
      - tele-conf:/srv
volumes:
  tele-conf:
```

This repository WILL contains a `demo` directory which hold the `docker-compose.yml` file as well as a `Makefile` which might come handy. From within the `demo` directory you can start the container simply by typing:


## Environment variables

When you create the `mlan/asterisk` container, you can configure the services by passing one or more environment variables or arguments on the docker run command line. Once the services has been configured a lock file is created, to avoid repeating the configuration procedure when the container is restated. In the rare event that want to modify the configuration of an existing container you can override the default behavior by setting `FORCE_CONFIG` to a no-empty string.

## Configuration files

Asterisk and its modules are configured using several configuration files which are typically found in `/etc/asterisk`. The  `/mlan/astisk` image provides a collection of configuration files which can serve as starting point for your system. We will outline how we intend the default configuration files are structured.

### Functional

Some of the collection of configuration files provided does not contain any user specific data and might initially be left unmodified. These files are:

| File name        | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| acl.conf         |                                                              |
| asterisk.conf    | asterisk logging, directory structure                        |
| ccss.conf        |                                                              |
| cli_aliases.conf | command line interface aliases convenience                   |
| extensions.conf  | dialplan how incoming and outgoing calls and messages are handled |
| features.conf    | activation of special features                               |
| indications.conf | dial tone local                                              |
| logger.conf      | logfiles                                                     |
| modules.conf     | activation of modules                                        |
| musiconhold.conf | music on hold directory                                      |
| pjproject.conf   | pjsip installation version                                   |

### Personal

| File name               | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| extensions-globals.conf | Defines SIP trunk endpoint                                   |
| minivm.conf             | Define mail sever URL and authentication credentials which voice mail email notifications will be sent |
| pjsip.conf              | Defines SIP transport, protocol, port, host URL              |
| pjsip_wizard.conf       | Defines endpoints, soft-phones, users, sip trunk             |
| rtp.conf                | Define RTP port range                                        |
| sms.conf                | Define HTTP SMS, incoming and outgoing                       |

### `pjsip_wizard.conf` soft-phones and trunks

### `extensions-globals.conf ` sms termination

### `pjsip.conf`,  `rtp.conf` network

### `minivm.conf` voice-mail



## Persistent storage

By default, docker will store the configuration and run data within the container. This has the drawback that the configurations and queued and quarantined mail are lost together with the container should it be deleted. It can therefore be a good idea to use docker volumes and mount the run directories and/or the configuration directories there so that the data will survive a container deletion.

To facilitate such approach, to achieve persistent storage, the configuration and run directories of the services has been consolidated to `/srv/etc` and `/srv/var` respectively. So if you to have chosen to use both persistent configuration and run data you can run the container like this:

```
docker run -d --name pbx-mta -v pbx-mta:/srv -p 127.0.0.1:25:25 mlan/asterisk
```



## Initialization procedure

The `mlan/asterisk` image is compiled without any configuration files. When a container is created using the `mlan/asterisk` image default configuration files are copied to the configuration directory `etc/asteroisk` if it is found to be empty. This behavior is intended to support the following initialization procedures.

In scenarios where you already have a collection of configuration files on a docker volume, start/create a `mlan/asterisk` container with this volume mounted. At startup these configuration files are recognized and left untouched and asterisk is stated. The same will happen when the container is restarted. 

In a scenario where we don't have any configuration files yet we start/create a want to start `mlan/asterisk` container with an empty target volume. At startup the default configuration files will be copied to the mounted volume. Now you can edit these configuration files to your liking either from within the container or directly from the volume mounting point on the  docker host. At consecutive startup these configuration files are recognized and left untouched and asterisk is stated.
