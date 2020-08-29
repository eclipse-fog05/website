---
title: "Hello World!"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To kick off our tour of Eclipse fog05 and to demonstrate use and access of the Fog Infrastructure Manager, we will start with the obligatory "hello world"
example.
This example will deploy a docker container that just prints "Hello World!"

Let's get started.

First, we need to install [containerd](https://containerd.io/) and the containerd plugin for Eclipse fog05


```bash
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/containerd.io_1.3.4-1_amd64.deb
$ wget https://github.com/eclipse-fog05/fog05/releases/download/v0.2.1/fog05-plugin-fdu-containerd_0.2.1-1_amd64.deb
$ sudo apt install ./containerd.io_1.3.4-1_amd64.deb
$ sudo apt install ./fog05-plugin-fdu-containerd_0.2.0-1.deb
```

Once it is installed let's start the fog05 services

```bash
$ sudo systemctl start containerd
$ sudo systemctl start zenoh
$ sudo systemctl start fos_agent
$ sudo systemctl start fos_linux
$ sudo systemctl start fos_linuxbridge
$ sudo systemctl start fos_ctd
```


Next we need to write the descriptor for this container.
Let's call it `fdu_helloworld.json`:

```json
{
    "id": "hello-world-docker",
    "name": "hello-world",
    "computation_requirements": {
        "cpu_arch": "x86_64",
        "cpu_min_freq": 0,
        "cpu_min_count": 1,
        "ram_size_mb": 64.0,
        "storage_size_gb": 0.1
    },
    "image": {
        "uri": "docker.io/gabrik91/loop:latest",
        "checksum": "",
        "format": ""
    },
    "storage": [],
    "hypervisor": "DOCKER",
    "migration_kind": "COLD",
    "interfaces": [],
    "io_ports": [],
    "connection_points": [],
    "depends_on": []
}
```


Let's imagine that our descriptor was saved in our `$HOME` directory,
we can use this simple python script to deploy the "hello world" example

```python
import json
import time
from fog05 import FIMAPI
from fog05_sdk.interfaces.FDU import FDU

def read_file(filepath):
	with open(filepath, 'r') as f:
		data = f.read()
	return data

api = FIMAPI()
desc = json.loads(read_file('/home/ubuntu/fdu_helloworld.json'))
fdu_descriptor =  FDU(desc)
fduD = api.fdu.onboard(fdu_descriptor)
fdu_id = fduD.get_uuid()
print ( 'fdu_id: {}'.format(fduD.get_uuid()))

inst_info=api.fdu.define(fdu_id)
api.fdu.configure(inst_info.get_uuid())
api.fdu.start(inst_info.get_uuid(), "MYENV=Hello World!")
print('Instance ID: {}'.format(inst_info.get_uuid()))
time.sleep(1)
log = api.fdu.log(inst_info.get_uuid())
print('FDU Output:\n{}\n'.format(log))

input('Press enter to terminate')

api.fdu.terminate(inst_info.get_uuid())

api.close()
exit(0)

```

We can save this file as `fos_deploy.py` and run it using `python3`

```bash
$ python3 fos_deploy.py
fdu_id: 6f52866c-0e69-4075-9ee1-b24c3b8a0969
Instance ID: 4ac683a6-7fee-4481-af2b-c7ca6c5bdf9c
FDU Output:
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!


Press enter to terminate
...
$
```


