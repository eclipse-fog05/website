---
title: "Write a Runtime Pluing"
weight: 7101
menu:
  docs:
    parent: going_deeper
---

# Runtime Plugin

The **Runtime** plugin implements the life-cycle management for *FDU*, is has to strictly follow the state machine defined for the *FDU* and provide handles for the state management and monitoring for the running *FDUs*.

This picture represent the state machine that an *FDU* has to follow.

![FDU State Machine](/img/fdufsm.png)

## States

Each *FDU* can be in one of those nine states:

- UNDEFINED
- DEFINED
- CONFIGURED
- RUNNING
- PAUSED
- SCALING
- MIGRATING
- TAKE_OFF
- LANDING

### UNDEFINED

In this state the FDU is registered in the catalog but there are no instances in the system

### DEFINED

In this state a FDU instance was created in a node, but it is only created, meaning that an instance UUID was assigned to it.

### CONFIGURE

The FDU instance was configured in a node, the image was fetched, resources allocated, and the instance was defined in the underlying hypervisor/container engine.
Or in case of native application the namespace was created.

### RUNNING

The FDU instance is started, the instance is started in the underlying hypervisor/container engine or the native application process is started.
Monitoring of the instance begin at this stage.

### MIGRATING

The FDU instance is instructed for migration, this leads to two different states on the source and destination node.
Source node instance transits to TAKE_OFF, when it is ready for migration, while destination instance transits to LANDING when ready for migration.
After migration the instance in the source node is destroyed, while in destination node the instance is in RUNNING state.
Migration is supported only for collaborative FDUs.

### PAUSE

The execution of the instance is freezed. This state is supported only for certain FDUs.

### SCALING

The instance is scaled up or down. This state is supported only for certain FDUs.


## State transitions

The *Runtime* plugin has to implement state transition for the FDU is able to manage.
The *Runtime* plugin can access to functionalities of the *OS* and *Network Manager* plugin using the `os` and `nm` objects.

State transitions are defined by the *RuntimePluginFDU* interface and by the implementation of the following functions.

### define_fdu
It takes as parameter the FDU instance record and create the instance in the node.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### undefine_fdu
Takes the instance UUID as parameter and destroy the instance from the node.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### configure_fdu
Takes the instance UUID as parameter and create all the resources needed for the execution of the FDU, creates the instance in the hypervisor/container engine if needed, creates the needed networking
components by calling the *Network Manager* plugin.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### clean_fdu
Takes the instance UUID as parameter and cleans the configured FDU, removes images, instance from hypervisor/container engine if needed, networking components, releases the resources.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### run_fdu
Take the instance UUID and starts the instance UUID, calls APIs from hypervisors/container engine in needed or start the native application.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### stop_fdu
Takes the instance UUID and stops the instance running, calls APIs from hypervisors/container engine in needed or send a signal the native application.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### pause_fdu
Takes the instance UUID and pauses the execution of the running FDU if it is possible.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### resume_fdu
Takes the instance UUID and resumes the execution of the paused FDU.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### scale_fdu
Takes the instance UUID and scaling parameters and scales the running FDU if it is possible.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`

### migrate_fdu
Takes the instance UUID and migrates the FDU from a source node to a destination node if it is possible.

In case of success updates the status using `update_fdu_status`

In case of error returns an error using `write_fdu_error`


---

The interface for a *Runtime* plugin is available in *Go* and *Python3* more implementation are coming soon.

See example implementations from the [LXD Plugin](https://github.com/eclipse/fog05/blob/master/fos-plugins/LXD/LXD_plugin)
