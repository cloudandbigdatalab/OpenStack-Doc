# UTSA OCI - Intel Internship
## OpenStack Magnum Project
## Shawn Aten, Annie Lezil, Mohan Muppidi

### 1. Overview
#### What does it do?
> Magnum is an OpenStack API service developed by the OpenStack Containers Team
making container orchestration engines such as Docker and Kubernetes available
as first class resources in OpenStack. Magnum uses Heat to orchestrate an OS
image which contains Docker and Kubernetes and runs that image in either
virtual machines or bare metal in a cluster configuration.

Magnum automates the setup of Kubernetes, Swarm, or Mesos clusters on OpenStack
deployments making setup faster and more consistent. The use of containers in
the cloud allows for denser utilization of VM and hardware resources.

#### Architecture
![Magnum Architecture Diagram](/magnum_architecture.png)

#### How does it work?
You create *bays*. A bay is where your container orchestrations software
(Kubernetes, Swarm, Mesos) runs. Bays can contain multiple nodes. A bay is
accessible to a single tenant. Bays with different engines can be run
side by side so you're not locked-in to a single platform. You utilize bays
through the each engine's standard client (kubectl, docker-swarm, etc.).

#### Interaction with other services
Magnum uses the standard OpenStack projects and Heat for orchestration.

> Magnum uses Heat to orchestrate an OS image which contains Docker and
Kubernetes [or Swarm or Mesos] and runs that image in either virtual machines
or bare metal in a cluster configuration.

![Screenshot of Heat Stack in Horizon](/magnum_heat.png)

Nodes of Magnum bays are normal Nova or Ironic instances.

#### CLI

### 2. Code Review, Code Tree Structure
*Not sure what's needed for this section.*  
Code Review is done through Gerrit.

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
