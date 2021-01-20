---
title: "FOrcE Installation"
weight : 2
menu:
  docs:
    parent: getting_started
---

Before getting started with Eclipse fog05 orchestation we need to install and [start](#start-eclipse-fog05-force) it

We have three possibilities for installing Eclipse fog05

- From [source](#from-source)
- From [debian packages](#from-debian-packages)
- Using a [docker image](#from-docker)



## From source



We can now clone the Eclipse fog05 repository and build the orchestation engine

```bash

$ git clone https://github.com/eclipse-fog05/fog05 -b 0.2.x
$ cd fog05/src/force
$ make
$ sudo install -m 0755 /usr/local/bin

```


## From debian packages

For each release `.deb` files are generated for Ubuntu 18.04 LTS, and works also with newer versions of Ubuntu. Those files can be found in the [release page on GitHub](https://github.com/eclipse-fog05/fog05/releases/tag/v0.2.2), and are available for `x86_64` and `aarch64`.


To install the `force` we can simply run

```bash

$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.2/fog05-force_0.2.2-1_amd64.deb
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.2/libzenoh-0.3.0-amd64.deb
$ sudo apt install ./libzenoh-0.3.0-amd64.deb ./fog05-force_0.2.2-1_amd64.deb

```

It will create the folder `/etc/fos` in which we can found the configuration file `force.json` and the `force`, and configures the systemd service `force` to start it.
The configuration file contains the address of the zenoh router used by the orchestrator.

## From docker

If you have docker and docker-compose enable in you machine you can download the [docker-compose.yaml](https://raw.githubusercontent.com/eclipse-fog05/fog05/0.2.x/src/force/docker-compose.yaml)

And deploy it using

```bash

$ wget https://raw.githubusercontent.com/eclipse-fog05/fog05/0.2.x/src/force/docker-compose.yaml
$ docker network create -d overlay fog05-forcenet
$ docker stack deploy -c docker-compose.yaml force

```


## Start Eclipse fog05 force

If installed by *source* if shall be started by hand

```bash

$ ZENOH=<ip address of a dedicated zenoh router> force

```

If installed by `.deb` file it can be started using systemd

To start:
```bash
$ sudo systemctl start force
```

To stop:
```bash

$ sudo systemctl stop force

```

Interaction is described in [fosctl usage]({{< ref "/docs/getting-started/introducing-fosctl" >}}).


Logs can be checked using `journalctl`, `journalctl -u force -f`

