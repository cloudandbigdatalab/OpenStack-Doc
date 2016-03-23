# OpenStack Magnum Project
UTSA OCI - Intel Internship  
Shawn Aten, Annie Lezil, Mohan Muppidi

## Introduction
> Magnum is an OpenStack API service developed by the OpenStack Containers Team making container orchestration engines such as Docker and Kubernetes available as first class resources in OpenStack. Magnum uses Heat to orchestrate an OS image which contains Docker and Kubernetes and runs that image in either virtual machines or bare metal in a cluster configuration.

Magnum automates the setup of Kubernetes, Swarm, or Mesos clusters on OpenStack deployments making setup faster and more consistent. The use of containers in the cloud allows for denser utilization of VM and hardware resources.

In magnum you create *bays*. A bay is where your container orchestrations software (Kubernetes, Swarm, Mesos) runs. Bays can contain multiple nodes. A bay is accessible to a single tenant. Bays with different engines can be run side by side so you're not locked-in to a single platform.

## Architecture
![Magnum Architecture Diagram](./photos/magnum_architecture.png)

Magnum uses the standard OpenStack projects and Heat for orchestration. Nodes of Magnum bays are normal Nova or Ironic instances.

> Magnum uses Heat to orchestrate an OS image which contains Docker and Kubernetes [or Swarm or Mesos] and runs that image in either virtual machines or bare metal in a cluster configuration.

![Screenshot of Heat Stack in Horizon](./photos/magnum_heat.png)

## Installation
Magnum can be installed in DevStack by enabling the plugin in your `local.conf`. `enable_plugin magnum https://git.openstack.org/openstack/magnum` The full details can be found [here](http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#exercising-the-services-using-devstack) The [Cloud Init](./cloud.init) file in this directory can be used to initialize a cloud machine with DevStack and Magnum.

*Details on full deployment coming soon.*

## CLI
Magnum is used through the *magnum* Python client. To interact with bays (create pods, run containers, etc.) you use the magnum client, not the kubectl or docker-swarm clients. Here's a partial overview of the available commands.

```
baymodel-create     Create a baymodel.
baymodel-delete     Delete specified baymodel.
baymodel-list       Print a list of bay models.
baymodel-show       Show details about the given baymodel.
baymodel-update     Updates one or more baymodel attributes.
bay-create          Create a bay.
bay-delete          Delete specified bay.
bay-list            Print a list of available bays.
bay-show            Show details about the given bay.
bay-update          Update information about the given bay.
ca-show             Show details about the CA certificate for a bay.
ca-sign             Generate the CA certificate for a bay.
container-create    Create a container.
container-delete    Delete specified containers.
container-exec      Execute command in a container.
container-list      Print a list of available containers.
container-logs      Get logs of a container.
container-pause     Pause specified containers.
container-reboot    Reboot specified containers.
container-show      Show details of a container.
container-start     Start specified containers.
container-stop      Stop specified containers.
container-unpause   Unpause specified containers.
service-list        Print a list of magnum services.
pod-create          Create a pod.
pod-delete          Delete specified pod.
pod-list            Print a list of registered pods.
pod-show            Show details about the given pod.
pod-update          Update information about the given pod.
rc-create           Create a replication controller.
rc-delete           Delete specified replication controller.
rc-list             Print a list of registered replication controllers.
rc-show             Show details about the given replication controller.
rc-update           Update information about the given replication
                    controller.
coe-service-create  Create a coe service.
coe-service-delete  Delete specified coe service.
coe-service-list    Print a list of coe services.
coe-service-show    Show details about the given coe service.
coe-service-update  Update information about the given coe service.
bash-completion     Prints arguments for bash-completion. Prints all of
                    the commands and options to stdout so that the
                    magnum.bash_completion script doesn't have to hard
                    code them.
help                Display help about this program or one of its
                    subcommands.
```

The Magnum Quick Start guide has instructions on [building a Kubernetes Bay](http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#using-kubernetes-bay) and [building a Swarm Bay](http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#building-and-using-a-swarm-bay).


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

## References
<https://wiki.openstack.org/wiki/Magnum>  
<http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#exercising-the-services-using-devstack>  
<http://docs.openstack.org/developer/devstack/guides/single-vm.html>  
<http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#using-kubernetes-bay>  
<http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html#building-and-using-a-swarm-bay>
