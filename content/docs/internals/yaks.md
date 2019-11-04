---
title: "Location transparent state access and distribution"
weight: 8002
menu:
  docs:
    parent: "internals"
---

Eclipse fog05 uses YAKS to provide location-transparent access to the system state
and to distribute this state.

# Distributed state

Eclipse fog05 is designed from the beginning to not rely in any *centralised* construct,
such as DB servers, controllers and so on.

We made this choice because the problem Eclipse fog05 is trying to address it the management
on the Fog infrastructure which by definition is distributed, heterogenous, mobile and with
intermittent connectivity. This environment has driven all the design choices in Eclipse fog05,
and so in very short time we found ourself with the problem to how access the system state in such environment.

As each *node* may have intermittent connectivity, can move from one place to another in the system, we needed
a way to provide **location-transparent** access to the state of each *node* as well as build the system state starting from
the state of each single *node*.

We moved towards build all the piece of software in a state-less fashion and leveraging the *location-transparency* state
access for storing and retrieving state information.

All the state information in Eclipse fog05 are stored in [YAKS](http://yaks.is) a distributed Key-value store, that support location-transparency access,
scalability, URI as Keys, it provides a *eventual consistency* model and mechanisms to trigger computations (and so build an RPC on top).

It leverages on the [Zenoh](http://zenoh.io) protocol to provide its functionalities.

# How Eclipse fog05 uses YAKS

In Eclipse fog05 all the components are designed to be state-less, and to store all the information in YAKS.
Information are organized following a tree structure in order to facilitate queries in YAKS.

## State separation

The whole system state is organized in two different portions.

- Public State
- Private State


Each of them is then separed in two different sub-portions.


- Actual State
- Desired State


As you can image from the name *Public* and *Private* stores respectively information that can be accessed globally and locally, while *Actual* and *Desired* stores respectively actual state information and set-point for the system.

Each of those portion have different read and write permissions, described in the following image.


![State permissions](/img/state_permission.png)


This separation allows to avoid write concurrency on the *Public Actual* state as only the *node* can write is own state, while on the *Public Desired* this is not an issue as the *node* will be able to order the requests. Each *node* owns only its state, the system state is built with the union of the *public* state of each *node* in the system.
As a consequence there is no *central* nodes and each *node* can act as entry point for the system.

This diagram shows how the *agent* and the *plugins* interact through YAKS.


![YAKS Interaction](/img/stores.png)


The *agent* listen to all the portion of the state, while *plugins* only listen the *private* portion, this portion is mainly used for agent-plugin communication, while the *public* one is used for inter-node and user-agent communication, the *agent* converts requests coming in the *public desired* portion to requests to the right plugin in the *private desired* portion.

As already described in the [architecture][arch] section, the YAKS server can be deployed in the same node of the *agent* or in a another one.


[arch]: {{< ref "/docs/going-deeper/architecture.md" >}}

## Tree structure

We said that the state is organized as a tree structure, this helps in the separation of the state of each *node* and in the definition on the storage and URIs to be used in YAKS.
There are a total of four different trees, organised in two groups.

- Public trees
- Private trees

Trees in the same group differs only for the first portion of the URI, while between different groups the tree structure is different.


Below you can find the strucutures

### Public

```
--- [a|d]gfos
    |--- <system id>
        |---- users
        |     |--- <user id>
        |          |--- info
        |
        |--- configuration
        |
        |--- info
        |
        |--- tenants
        |     |--- <tenant id>
        |          |--- info
        |          |--- configuration
        |          |--- catalog
        |          |    |--- fdu
        |          |    |    |--- <fdu id>
        |          |    |         |--- info
        |          |    |
        |          |    |--- entity
        |          |    |     |--- <entity id>
        |          |                |--- info
        |          |
        |          |--- flavor
        |          |     |--- <flavor id>
        |          |            |--- info
        |          |
        |          |--- image
        |          |    |--- <image id>
        |          |           |--- info
        |          |
        |          |--- storage
        |          |     |--- <storage id>
        |          |            |--- info
        |          |
        |          |--- networks
        |          |     |--- <network id>
        |          |     |      |--- info
        |          |     |
        |          |     |--- ports
        |          |     |      |--- <port id>
        |          |     |             |--- info
        |          |     |
        |          |     |--- routers
        |          |            |--- <router id>
        |          |                   |--- info
        |          |
        |          |--- nodes
        |          |     |--- <node id>
        |          |            |--- agent
        |          |            |     |--- exec
        |          |            |            |--- functions...
        |          |            |
        |          |            |--- info
        |          |            |--- status
        |          |            |--- configuration
        |          |            |--- plugins
        |          |            |     |--- <plugin id>
        |          |            |     |       |--- info
        |          |            |     |       |--- exec
        |          |            |                   |--- functions ...
        |          |            |--- flavor
        |          |            |      |--- <flavor id>
        |          |            |             |--- info
        |          |            |
        |          |            |--- image
        |          |            |     |--- <image id>
        |          |            |            |--- info
        |          |            |
        |          |            |--- storage
        |          |            |     |--- <storage id>
        |          |            |            |--- info
        |          |            |
        |          |            |--- fdu
        |          |            |    |--- <fdu id>
        |          |            |            |--- instances
        |          |            |                   |--- <instance id>
        |          |            |                           |--- info
        |          |            |
        |          |            |--- networks
        |          |            |     |--- <network id>
        |          |            |     |      |--- info
        |          |            |     |
        |          |            |     |--- ports
        |          |            |     |       |--- <port id>
        |          |            |     |             |--- info
        |          |            |     |
        |          |            |     |--- routers
        |          |            |     |     |--- <router id>
        |          |            |     |             |--- info
        |          |            |     |
        |          |            |     |--- floating-ips
        |          |            |     |     |--- <floating ip id>
        |          |            |     |             |--- info

```



### Private

```
--- [a|d]lfos
        |--- <node id>
        |      |--- agent
        |      |     |--- exec
        |      |          |--- functions...
        |      |
        |      |--- info
        |      |--- status
        |      |--- configuration
        |      |--- plugins
        |      |     |--- <plugin id>
        |      |     |     |--- info
        |      |     |--- exec
        |      |           |--- functions ...
        |      |
        |      |--- runtimes
        |      |      |--- <plugin id>
        |      |            |--- flavor
        |      |            |     |--- <flavor id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- image
        |      |            |      |--- <image id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- storage
        |      |            |     |--- <storage id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- fdu
        |      |            |    |--- <fdu id>
        |      |            |            |--- instances
        |      |            |                   |--- <instance id>
        |      |            |                           |--- info
        |      |
        |      |--- network_managers
        |      |      |--- <plugin id>
        |      |            |--- networks
        |      |            |     |--- <network id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- ports
        |      |            |     |--- <port id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- routers
        |      |            |      |--- <router id>
        |      |            |             |--- info
        |      |            |
        |      |            |--- floating-ips
        |      |            |     |--- <floating ip id>
        |      |            |            |--- info
        |      |            |
        |      |            |--- exec
        |      |            |     |--- functions ...
```


The usage of this structure is implemented in the `YaksConnector` object implemented in the SDK for different languages.