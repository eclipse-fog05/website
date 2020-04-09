---
title: "Installation"
weight : 1
menu:
  docs:
    parent: getting_started
---

Before getting started with Eclipse fog05 we need to install and [start](#start-eclipse-fog05) it

We have two possibilities for installing Eclipse fog05

- From [source](#from-source)
- From [debian packages](#from-debian-packages)



## From source


#### Agent

Eclipse fog05 core component is written in OCaml, hence to build it we need
a functional OCaml environment, luckily installing OCaml compiler and build system is not that complicated.



```bash
$ sudo apt install jq libev-dev libssl-dev m4 pkg-config rsync unzip bubblewrap -y
$ sh <(curl -sL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)
$ opam init -c 4.09.0
$ eval $(opam env)
```

After that we can install all the OCaml dependencies:

```bash
$ opam install dune.1.11.4 atdgen.2.0.0 conf-libev ocp-ocamlres -y
$ opam pin add apero-core https://github.com/atolab/apero-core.git#0.4.6 -y
$ opam pin add dynload-sys https://github.com/atolab/apero-core.git#0.4.6 -y
$ opam pin add apero-net https://github.com/atolab/apero-net.git#0.4.6 -y
$ opam pin add apero-time https://github.com/atolab/apero-time.git#0.4.6 -y
$ opam pin add zenoh-proto https://github.com/atolab/zenoh.git#0.3.0 -y
$ opam pin add zenoh-ocaml https://github.com/atolab/zenoh.git#0.3.0 -y
$ opam pin add yaks-common https://github.com/atolab/yaks-common.git#0.3.0 -y
$ opam pin add yaks-ocaml https://github.com/atolab/yaks-ocaml.git#0.3.0 -y
$ opam pin add fos-sdk https://github.com/eclipse-fog05/sdk-ocaml.git#master -y
$ opam pin add fos-fim-api https://github.com/eclipse-fog05/api-ocaml.git#master  -y

```

Then we need to get the Eclipse Zenoh daemon, fog05 uses version 0.3.0 building instruction can be found [here](https://github.com/atolab/fog05_debs/blob/master/zenoh/generate_deb.sh)


We can now clone the Eclipse fog05 agent repository and build and install the agent

```bash
$ git clone https://github.com/eclipse-fog05/agent
$ cd agent
$ make
$ sudo make install
```

The installation creates the folder `/etc/fos` in which we can found the configuration file `agent.json` and the `agent` itself. It creates as well the systemd files for starting and stopping the service `fos_agent`


After installing the agent is required to install at least the OS, Network Manager and one FDU plugin.

#### Linux Plugin

To install the OS plugin, we need Python3 and a set of dependencies


```bash
$ sudo apt install python3 python3-dev python3-pip cmake build-essential
$ git clone https://github.com/atolab/zenoh-c -b 0.3.0 && cd zenoh && make && sudo make install && cd ..
$ pip3 install yaks==0.3.0 zenoh==0.3.0 psutil netifaces pyangbind sphinx jinja2 packaging
$ git clone https://github.com/eclipse-fog05/sdk-python cd sdk-python && make && sudo make install && cd ..

```

Then we can clone and install the Linux Plugin

```bash
$ git clone https://github.com/eclipse-fog05/plugin-os-linux
$ cd plugin-os-linux
$ sudo make install
```

After the installation the directory `/etc/fos/plugins/plugin-os-linux` is created and the plugin configuration `/etc/fos/plugins/plugin-os-linux/linux_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_linux` is created.


#### Linux Bridge Plugin

To install the Network Manager plugin, we need the same dependencies as the Linux one

So we can clone and install the Network Manager Plugin

```bash
$ git clone https://github.com/eclipse-fog05/plugin-net-linuxbridge
$ cd plugin-os-linux
$ sudo make install
```

After the installation the directory `/etc/fos/plugins/plugin-net-linuxbridge` is created and the plugin configuration `/etc/fos/plugins/plugin-net-linuxbridge/linuxbridge_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_linuxbridge` is created.


#### FDUs Plugin


For this guide we show how to install the LXD plugin, the steps are similar for the others plugins.

We install the dependencies

```bash
sudo pip3 install pylxd
sudo snap install lxd
```

Then we clone and install the plugin


```bash
$ git clone https://github.com/eclipse-fog05/plugin-fdu-lxd
$ cd plugin-os-linux
$ sudo make install
```

After the installation the directory `/etc/fos/plugins/plugin-fdu-lxd` is created and the plugin configuration `/etc/fos/plugins/plugin-fdu-lxd/LXD_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_lxd` is created.


## From debian packages

For each release `.deb` files are generated for latest Ubuntu LTS and Debian. Those files can be found in the [release page on GitHub](https://github.com/eclipse-fog05/fog05/releases/tag/v0.1.0), and are available for `x86_64` and `aarch64`.

#### Agent

To install the `agent` we can simply run

```bash
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/zenoh_0.1.0-1_amd64-ubuntu.deb
sudo apt install ./zenoh_0.1.0-1_amd64-ubuntu.deb
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/fog05_0.1.0-1_amd64_ubuntu-bionic.deb
$ sudo apt install ./fog05_0.1.0-1_amd64_ubuntu-bionic.deb
```

It will create the folder `/etc/fos` in which we can found the configuration file `agent.json` and the `agent`, and configures the systemd service `fos_agent` and the Zenoh systemd service `zenoh`

#### Linux Plugin

To install the Linux Plugin we can simply run

```bash
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/libzenoh-0.3.0-ubuntu-amd64.deb
$ sudo apt install ./libzenoh-0.3.0-ubuntu-amd64.deb
$ sudo pip3 install yaks==0.3.0 zenoh==0.3.0
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/python3-fog05-sdk_0.1.0-1_all.deb
sudo apt install ./python3-fog05-sdk_0.1.0-1_all.deb
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/fog05-plugin-os-linux_0.1.0-1_amd64_ubuntu_bionic.deb
$ sudo apt install ./fog05-plugin-os-linux_0.1.0-1_amd64_ubuntu_bionic.deb
```

After the installation the directory `/etc/fos/plugins/plugin-os-linux` is created and the plugin configuration `/etc/fos/plugins/plugin-os-linux/linux_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_linux` is created.

#### Linux Bridge Plugin

To install the LinuxBridge Plugin we can simply run

```bash
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/fog05-plugin-net-linuxbridge_0.1.0-1_amd64_ubuntu_bionic.deb
$ sudo apt install ./fog05-plugin-net-linuxbridge_0.1.0-1_amd64_ubuntu_bionic.deb
```

After the installation the directory `/etc/fos/plugins/plugin-net-linuxbridge` is created and the plugin configuration `/etc/fos/plugins/plugin-net-linuxbridge/linuxbridge_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_linuxbridge` is created.

#### FDUs plugin

For this guide we show how to install the LXD plugin, the steps are similar for the others plugins.

We install the dependencies

```bash
sudo snap install lxd
```

Then we install the plugin

```bash
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.1.0/fog05-plugin-fdu-lxd_0.1.0-1_amd64_ubuntu-bionic.deb
$ sudo apt install ./fog05-plugin-fdu-lxd_0.1.0-1_amd64_ubuntu-bionic.deb
```

After the installation the directory `/etc/fos/plugins/plugin-fdu-lxd` is created and the plugin configuration `/etc/fos/plugins/plugin-fdu-lxd/LXD_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_lxd` is created.

## Start Eclipse fog05


Eclipse fog05 components can be started and stopped using systemd.

To start:
```bash
sudo systemctl start zenoh
sudo systemctl start fos_agent
sudo systemctl start fos_linux
sudo systemctl start fos_linuxbridge
sudo systemctl start fos_lxd
```

To stop:
```bash
sudo systemctl stop fos_lxd
sudo systemctl stop fos_linuxbridge
sudo systemctl stop fos_linux
sudo systemctl stop fos_agent
sudo systemctl stop zenoh
```


Logs can be checked using `journalctl`, `journalctl -u <fog05 component> -f`

