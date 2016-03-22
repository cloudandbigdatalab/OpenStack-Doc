# UTSA OCI - Intel Internship
# OpenStack Magnum Project
# Shawn Aten, Annie Lezil, Mohan Muppidi

## Introduction
> Magnum is an OpenStack API service developed by the OpenStack Containers Team making container orchestration engines such as Docker and Kubernetes available as first class resources in OpenStack. Magnum uses Heat to orchestrate an OS image which contains Docker and Kubernetes and runs that image in either virtual machines or bare metal in a cluster configuration.

Magnum automates the setup of Kubernetes, Swarm, or Mesos clusters on OpenStack deployments making setup faster and more consistent. The use of containers in the cloud allows for denser utilization of VM and hardware resources.

In magnum you create *bays*. A bay is where your container orchestrations software (Kubernetes, Swarm, Mesos) runs. Bays can contain multiple nodes. A bay is accessible to a single tenant. Bays with different engines can be run side by side so you're not locked-in to a single platform. You utilize bays through the each engine's standard client (kubectl, docker-swarm, etc.).

## Architecture
![Magnum Architecture Diagram](./photos/magnum_architecture.png)

Magnum uses the standard OpenStack projects and Heat for orchestration. Nodes of Magnum bays are normal Nova or Ironic instances.

> Magnum uses Heat to orchestrate an OS image which contains Docker and Kubernetes [or Swarm or Mesos] and runs that image in either virtual machines or bare metal in a cluster configuration.

![Screenshot of Heat Stack in Horizon](./photos/magnum_heat.png)

## Installation
Magnum can be installed in DevStack by enabling the plugin in your `local.conf`. `enable_plugin magnum https://git.openstack.org/openstack/magnum` The full details can be found [here](http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#exercising-the-services-using-devstack) The [Cloud Init](./cloud.init) file in this directory can be used to initialize a cloud machine with DevStack and Magnum.

*Details on full deployment coming soon.*

## CLI

## Code Review
*Not sure what's needed for this section.*  
Code Review is done through Gerrit.

## Code Contribution
```
magnum/
|-- contrib
|   `-- templates
|       `-- example
|           `-- example_template
|-- devstack
|   `-- lib
|-- doc
|   `-- source
|       |-- dev
|       `-- images
|-- etc
|   `-- magnum
|-- magnum
|   |-- api
|   |   |-- controllers
|   |   |   `-- v1
|   |   `-- middleware
|   |-- cmd
|   |-- common
|   |   |-- cert_manager
|   |   |-- pythonk8sclient
|   |   |   |-- swagger_client
|   |   |   |   |-- apis
|   |   |   |   `-- models
|   |   |   `-- templates
|   |   `-- x509
|   |-- conductor
|   |   |-- handlers
|   |   |   `-- common
|   |   `-- tasks
|   |-- db
|   |   `-- sqlalchemy
|   |       `-- alembic
|   |           `-- versions
|   |-- hacking
|   |-- locale
|   |   `-- ja
|   |       `-- LC_MESSAGES
|   |-- objects
|   |-- public
|   |   `-- css
|   |-- service
|   |-- servicegroup
|   |-- templates
|   |   |-- kubernetes
|   |   |   |-- elements
|   |   |   |   `-- kubernetes
|   |   |   `-- fragments
|   |   |-- mesos
|   |   |   |-- elements
|   |   |   |   |-- docker
|   |   |   |   |   |-- post-install.d
|   |   |   |   |   `-- pre-install.d
|   |   |   |   `-- mesos
|   |   |   |       |-- post-install.d
|   |   |   |       `-- pre-install.d
|   |   |   `-- fragments
|   |   `-- swarm
|   |       `-- fragments
|   `-- tests
|       |-- contrib
|       |-- functional
|       |   |-- api
|       |   |   `-- v1
|       |   |       |-- clients
|       |   |       `-- models
|       |   |-- common
|       |   |-- k8s
|       |   |-- mesos
|       |   |-- swarm
|       |   `-- tempest_tests
|       `-- unit
|           |-- api
|           |   `-- controllers
|           |       `-- v1
|           |-- common
|           |   |-- cert_manager
|           |   `-- x509
|           |-- conductor
|           |   |-- handlers
|           |   |   `-- common
|           |   `-- tasks
|           |-- db
|           |   `-- sqlalchemy
|           |-- objects
|           |-- service
|           |-- servicegroup
|           `-- template
|-- specs
`-- tools
```
