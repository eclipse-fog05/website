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

Eclipse fog05 is written in rust, hence to build it we need
a functional rust environment, luckily installing rust compiler and build system is not that complicated.



```bash

$ sudo apt install build-essential -y
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o /tmp/rust.sh && chmod +x /tmp/rust.sh
$ /tmp/rust.sh --default-toolchain nightly -y

```

We can now clone the Eclipse fog05 repository and build the agent

```bash

$ git clone https://github.com/eclipse-fog05/fog05
$ cd fog05
$ cargo build --release --all-targets

```
If we are running on apt-based system (like Debian or Ubuntu), we can ask `cargo` to build an installation package for us.

```bash

$ cargo deb -p fog05-agent
$ cargo deb -p fog05-fosctl

```

Then we can install those packages using classical `apt` commands.

```bash

$ sudo apt install ./target/debian/fog05-agent_0.3.0~alpha1_amd64.deb ./target/debian/fog05-fosctl_0.3.0~alpha1_amd64.deb

```

The installation creates the folder `/etc/fos` in which we can found the configuration file `agent.yaml` and the `fog05-agent` itself. It creates as well the systemd files for starting and stopping the service `fos-agent`

We then need to configure our agent, we can open the `/etc/fos/agent.yaml` file with an editor of our choice and set the `mgmt_interface` according to our configuration. Usually your first interface will be file.






After installing the agent is required to install at least the  Networking plugin and one Hypervisor plugin.


#### Linux Bridge Networking Plugin

To install the Network Manager plugin, we start by cloning its repository.

```bash

$ git clone https://github.com/eclipse-fog05/fog05-networking-linux
$ cd fog05-networking-linux
$ cargo build --release --all-targets

```

If we are running on apt-based system (like Debian or Ubuntu), we can ask `cargo` to build an installation package for us.

```bash

$ cargo deb

```

Then we can install the package using classical `apt` commands.

```bash

$ sudo apt install ./target/debian/fog05-networking-linux_0.3.0~alpha1_amd64.deb

```

After the installation the directory `/etc/fos/linux-network` is created and the plugin configuration `/etc/fos/linux-network/config.yaml` and the systemd service `fos-net-linux` is created.

We then need to configure the plugin, we can open the `/etc/fos/linux-network/config.yaml` file with an editor of our choice and set both `overlay_iface` and `dataplane_iface` according to our network configuration.
Usually your first interface will be file.


#### Hypervisors Plugin


For this guide we show how to install the Native plugin, the steps are similar for the others plugins.

We start by cloning the repository


```bash

$ git clone https://github.com/eclipse-fog05/fog05-hypervisor-native
$ cd fog05-hypervisor-native
$ cargo build --release --all-targets

```
If we are running on apt-based system (like Debian or Ubuntu), we can ask `cargo` to build an installation package for us.

```bash

$ cargo deb

```

Then we can install the package using classical `apt` commands.

```bash

$ sudo apt install ./target/debian/fog05-hypervisor-native_0.3.0~alpha1_amd64.deb

```

After the installation the directory `/etc/fos/native-hypervisor` is created and the plugin configuration `/etc/fos/native-hypervisor/config.yaml` and the systemd service `fos-hv-native` is created.


## From debian packages

{{% notice warning %}}
| Installation from debian packages installs the 0.2.2 version.
| This is the old OCaml based version that WILL NOT receive any bugfix or support.
{{% /notice %}}

For each release `.deb` files are generated for Ubuntu 18.04 LTS, and works also with newer versions of Ubuntu. Those files can be found in the [release page on GitHub](https://github.com/eclipse-fog05/fog05/releases/tag/v0.2.1), and are available for `x86_64` and `aarch64`.

#### Agent

To install the `agent` we can simply run

```bash

$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/zenoh_0.3.0-1_amd64.deb
sudo apt install ./zenoh_0.3.0-1_amd64.deb
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/fog05_0.2.1-1_amd64.deb
$ sudo apt install ./fog05_0.2.0-1_amd64.deb

```

It will create the folder `/etc/fos` in which we can found the configuration file `agent.json` and the `agent`, and configures the systemd service `fos_agent` and the Zenoh systemd service `zenoh`

#### Linux Plugin

To install the Linux Plugin we can simply run

```bash

$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/libzenoh-0.3.0-Linux.deb
$ sudo apt install ./libzenoh-0.3.0-Linux.deb
$ sudo pip3 install fog05-sdk==0.2.1 zenoh==0.3.0 yaks==0.3.0.post1
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/fog05-plugin-os-linux_0.2.1-1_amd64.deb
$ sudo apt install ./fog05-plugin-os-linux_0.2.1-1_amd64.deb

```

After the installation the directory `/etc/fos/plugins/plugin-os-linux` is created and the plugin configuration `/etc/fos/plugins/plugin-os-linux/linux_plugin.json` is populated with the `/etc/machine-id` as `nodeid` value. The systemd service `fos_linux` is created.

#### Linux Bridge Plugin

To install the LinuxBridge Plugin we can simply run

```bash

$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/fog05-plugin-net-linuxbridge_0.2.1-1_amd64.deb
$ sudo apt install ./fog05-plugin-net-linuxbridge_0.2.1-1_amd64.deb

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

$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/fog05-plugin-fdu-lxd_0.2.1-1_amd64.deb
$ sudo apt install ./fog05-plugin-fdu-lxd_0.2.1-1_amd64.deb

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

