---
title: "Architecture"
weight: 7100
menu:
  docs:
    parent: going_deeper
---

Eclipse fog05 is divided in two modules:

 - Fog Orchestration Engine (**FOrcE**)
 - Fog Infrastrucure Manager (**FIM**)

 Both modules provides different abstractions an APIs.

---

# Fog Infrastrucutre Manager


This module **virtualises** the **hardware** infrastructure, such as
**computational**, **communication**, **storage** and **I/O resources**.

It provides key primitives for the management of the **infrastructure** and
the application deployed in it.

It also provide a set of abstraction.

The **FIM** is designed to be distributed, without any *controller* node. The system state is managed in a location-transparent manner
using a distributed Key-value store, namely [YAKS](http://yaks.is), and leverages on [Zenoh](http://zenoh.io) for data distribution and sharing across the system.

## Abstractions

This section describes the *abstractions* provided by the *FIM*.

### Resources

In Eclipse fog05 a **resource** is any kind of assett that can be assigned either in a shared or exclusive manner.

The different kinds of resources supported by fog05 are **computational resources**, such as, CPU, FPGA, GPU,
**storage resources**, such as, RAM, block storage, object storage, **networking resources**, such as, 802.11 devices, tunnels and
**I/O resources**, such as, COM, CAN, GPIO and I2C.

### Node

An Eclipse fog05 **node** represents a locus of resources.
Examples of fog05 nodes are: server, an RPi, an embedded board.

In order to expose its resources in the system each node has either a running fog05 agent or a device pluigin.


### Network

In Eclipse fog05 a **network** represent a level-2 or level-3 network between a collection of deployable components.

Such networks are usually virtualized, and span across different nodes, even end-to-end, they can also be attached to
physical networks.


### Router

An Eclipse fog05 **router** is a networking element able to interconnect two or more networks.

It can also connect a network to the physical networking infrastructure.


### Port

A **port** in Eclipse fog05 is used to represent a network connection between a deployable unit and a network.

Ports are also present in router to connect the router to the different networks, a port can have a floating IP assigned to it.

### Fog Deployment Unit (FDU)

In Eclipse fog05 a **Fog Deployment Unit**, abbreviated to **FDU**, is **an indivisible unit of deployement**, such as, a binary executable, a Unikernel,
a container or a virtual machine.

An FDU **requires** a certain kind of **resources** as a precondition to its execution.

The life-cycle of an FDU is defined by a Finite State Machine.

## Components

The is composed by the following components.

### Agent

The **agent** is the component in charge of the closed-loop management for a node and the advertising of node resources and functionalities to the whole system.

It also provides an entry point for the node management and monitoring as well as the management of the FDUs running on a node.


### Plugins

The FIM is designed to leverages plugins for most of its functionalities, in Eclipse fog05 different kinds of plugin can be used.

#### OS Plugin

The **OS** plugin is the one used to abstract the underlying operating system and provide primitives to *agent* and other plugins.
OS plugins have to implement an interface in order to be recognized by fog05.

In Eclipse fog05 the following *OS* plugin are implemented:

- Linux
- Windows

#### Network Manager Plugin

The **Network Manager** plugin is used to create all the network related components, such as, *network*, *routers* and *ports*, is has to provide an implementation for each
of this component. This plugin provides also some functionalities to *agent* and other plugins.
Network Manager plugins have to implement an interface in order to be recognized by fog05.

In Eclipse fog05 the following *Network Manager* plugin are implemented:

- Linux bridge with VxLAN virtual networks, and routers in isolated network namespaces.

#### Runtime Plugin

The **Runtime** plugin implements the life-cycle management for *FDU*, is has to strictly follow the state machine defined for the *FDU* and provide handles for the state management and monitoring for the running *FDUs*.
Runtime plugins have to implement an interface in order to be recognized by fog05.

In Eclipse fog05 the following *Runtime* plugin are implemented:

- KVM Virtual Machine
- Linux Containers (LXD)
- Native applications (Linux and Windows)

The following ones are going to be implemented soon:

- containerd (docker) containers
- ROS2 Robotic framework


#### Device Plugin

This kind of plugin is the one that abstract the three previous one, and it is designed to allow the deployment and discovery of micro-controller and IoT board that does not have an operating system.

The *Device* plugin has to provide a subset of the functionalities provided by *OS*, *Network Manager* and *Runtime* plugin. It will also be connected and managed by an *agent* running on a different node.  For the user point-of-view, the resource constrained device look like any other *node* of the system.
It is difficult to provide a real interface to be implemented for the device plugin, we plan to provide an example implementation for a micro-controller that can be used as base for other implementations.



## Deployment

All the component of Eclipse fog05 are designed to be deployed in a distributed manner, in order to address the system heterogeneity.
In particular the minimal set to run a fog05 *node* is to have an *agent* and a set of *plugins* running on it, while in the case of micro-controllers the minimal set is composed by only the *device* plugin. Other components such as the YAKS server and Zenoh routers can be either deployed in a fog05 *node* or on different machines.

This *composable* deployment allows to reduce the footprint and optimize the resource utilization of the infrastructure  software.


![Deployment examples](/img/deployments.png)

---

# Fog Orchestation Engine

The **Fog Orchestration Engine** is responsible for the end-to-end orchestration and management of applications and services in the Fog infrastructure.

It enforces the **constraints** defined by the application/service and provides an allocation algorithm for the components of an application/service.

This component is still in the design phase, it also leverages on state distribution and location-transparent data access for its functionalities.