# OpenStack Configurator for Academic Research (OSCAR)

### Introduction

OpenStack is the open source software for creating public and private clouds. OpenStack software controls large pools of compute, storage, and networking resources throughout a datacenter, managed through a dashboard or via the OpenStack API. OpenStack works with popular enterprise and open source technologies making it ideal for heterogeneous infrastructure \[[1]\].

OpenStack has a modular architecture with various code names for its components. Compute (cloud fabric controller - Nova), image service (Glance), dashboard (Horizon), networking (Neutron), block storage (Cinder), object storage (Swift) and identity service (Keystone) are some important components. OpenStack can be setup with mix and match of different components. Several organizations have their own configuations for installing OpenStack, OpenStack-Ansible is one of them. OpenStack-Ansible is an official OpenStack project which aims to deploy production environments from source in a way that makes it scalable while also being simple to operate, upgrade, and grow \[[2]\].

OpenStack setup in multi-node configuration is non trivial and requires proper configuration of computing resources. In academia provisioning these resources is arduous. Due to this complex nature of OpenStack installation academic research in this field has not advanced.

The OSCAR project is designed to simplify the configuration of resources and OpenStack installation process. This project has scripts to configure the computing resources and prepare them for OpenStack deployment using OpenStack-Ansible, an official OpenStack project. This package also pulls stable version of the OpenStack-Ansible from github and edits the configuration files to match the configured environment.

### Overview

<!--![Components of cluster](https://github.com/UTSA-OCI/OCI-OpenStack-Ansible/blob/master/Docs/Figures/Slide6.jpg "Components of cluster" )-->
<img src="https://github.com/cloudandbigdatalab/OSCAR/blob/gh-pages/assets/figures/Slide6.jpg?raw=True" width="100%">

This guide gives a walk through OpenStack deployment on 5 nodes cluster (1 Controller node and 4 Compute nodes). The Controller and Compute nodes will have different components installed in them as shown in figure.

***Controller node:*** As the name suggests, this node controls the OpenStack cluster. This contains all the important components for proper functioning of the OpenStack cluster. Components like identity service, image service, networking and compute are installed in this node.

***Compute node:*** The virtual machines are hosted in these nodes. The compute service, hypervisor and network agents are installed in this node.

<!--![Components of cluster](https://github.com/UTSA-OCI/OCI-OpenStack-Ansible/blob/master/Docs/Figures/Slide4.jpg "Components of cluster" )-->
<img src="https://github.com/cloudandbigdatalab/OSCAR/blob/gh-pages/assets/figures/Slide4.jpg?raw=True" width="100%">

The deployment process is 4 fold as mentioned below.

- Servers setup
- Pre deployment
- OpenStack deployment
- Post deployment







<!--

- Servers setup and requirements
- Pre deployment instructions
- OpenStack deployment
- Post deployment instructions

-->





[1]: https://www.openstack.org
[2]: https://github.com/openstack/openstack-ansible

#### System requirements

We strongly recommend using Ubuntu 14.04 as operating system for the servers. Openstack deployment using this project is developed and tested for usage with Ubuntu 14.04.   

#### Provisioning a bare metal server using chameleon cloud

 Chameleon cloud users may have to add an Ubuntu 14.04 image before they spin the servers. This process can be found here <Link for adding a new bare metal image>. For additional informtion on usage of chameleon cloud please visit www.chameleoncloud.org.

#### Key pair authentication

OSCAR and OpenStack-ansible projects run based on ansible. For using ansible with the cluster the controller node has to have access to all the nodes along with itself. The access to the controller nodes can be granted using key pairs. Once the servers are up and running, create a key pair on controller node add the public key to authorized_keys of all the ***all nodes along with controller node itself***. Here are the set of instructions to do that.

Create a key pair on controller node. Make sure that you are logged in as root user while performing these steps.

```
 ssh-keygen -f .ssh/id_rsa -N ""  
```

This command should have created two files ``` id_rsa ``` and ``` id_rsa.pub ``` in ``` .ssh/ ``` folder. These files are called as a key-pair. ``` id_rsa ``` is called as private key and ``` id_rsa.pub ``` is called as a public key.

##### Adding the public keys to the authorized_keys of root user on all nodes including controller node

Adding the public key into authorized_keys file can be done in two ways.

   - Using ``` ssh-copy-id ```

   - Manually adding the public key

##### Using ``` ssh-copy-id ```
This method is simple and works fine when the root user of the nodes can be accessed using password. Use the following command

```
ssh-copy-id root@<host ip>
```

Where host-ip should be replaced by ip address of all the servers including controller node itself.

##### Manually adding the public key
In case of Chameleon cloud, password authentication is disabled by default. And if you can access the root user on the node only using private key then follow this method.

On your controller node open the open ``` ~/.ssh/id_rsa.pub ```

```
cat ~/.ssh/id_rsa.pub
```

ssh to the node and change to root user,

```
ssh userid@<node-ip>
sudo -i
```

Now, open the `~/.ssh/authorized_keys` file using nano or vim and add the copied contents,the public key, to the file and save it. Make sure this is done for all the nodes.



Check if all the nodes can be accessed using ``` ssh root@<node-ip> ``` command. Once controller node has access to all the nodes, the setup is ready for pre deployment procedures.

#### Update and Install git
Its time to clone OSCAR repo, lets update the apt-packages lists and install git

```
apt-get update
apt-get install git -y
```

### Pre-deployment setup

#### Cloning the git repository
OCI-OpenStack-Ansible has to be cloned into ```/opt/``` directory. Create ```/opt``` directory if it doesn't exists and change into that directory

```
mkdir -p /opt
cd /opt
```

Clone the github repo of OSCAR and enter the OSCAR directory

```
git clone https://github.com/cloudandbigdatalab/OSCAR.git

cd OSCAR
```

#### Configuring the OSCAR config file

Create ```/etc/oscar``` folder and copy ```oscar.conf.example``` to this folder as ```oscar.conf```

```
mkdir /etc/oscar

cp /opt/OSCAR/oscar.conf.example /etc/oscar/oscar.conf
```

In this directory there is an config which should have the information of all the hosts. Open oscar.conf file with your favorite editor

```
nano /etc/oscar/oscar.conf
```

The file looks like this

```
network:
  container_network: 172.17.100.0/24
  tunnel_network: 172.17.101.0/24
  storage_network: 172.17.102.0/24
  vm_gateway: 172.17.248.1

nodes:
  controller: 10.20.109.72
  computes:
    - 10.20.109.78
    - 10.20.109.79
    - 10.20.109.80

```

Add the ip address of controller and compute hosts at appropriate section as shown above and save the file.

#### Installing Ansible

Ansible is required to deploy OpenStack using OSCAR project. There is a bootstrap script available in /opt/OSCAR/scripts directory which sets the ansible up for the user.

```
cd /opt/OSCAR/scripts
./bootstrap-ansible.sh

```

#### Preparing the environment for OpenStack deployment

<!--![Architecture](https://github.com/UTSA-OCI/OCI-OpenStack-Ansible/blob/master/Docs/Figures/Slide1.jpg "Architecture" )-->
<img src="https://github.com/cloudandbigdatalab/OSCAR/blob/gh-pages/assets/figures/Slide7.jpg?raw=True" width="100%">

The project comes with an set of ansible playbooks which can prep the controller and compute hosts environment for Openstack deployment using openstack-ansible. These playbooks also edit some configuration files from the original openstack-ansible project to make those scripts compatible with the environment created. Go ahead an run the following command from /opt/OSCAR to start configuring the controller and compute hosts.

```
cd /opt/OSCAR
ansible-playbook bootstrap-openstack-play.yml
```

The above command should have cloned openstack-ansible repo in /opt directory and changed some configuration files to suit the environment created.

#### Enabling scp_if_true in /opt/openstack-ansible/playbooks/ansible.cfg

Open /opt/openstack-ansible/playbooks/ansible.cfg using your favorite editor

```
nano /opt/openstack-ansible/playbooks/ansible.cfg
```

add ```scp_if_true=True``` below ```[ssh_connection]``` section as shown below

```
[defaults]
gathering = smart

#hostfile = chameleon_cloud_node
hostfile = inventory
host_key_checking = False

# Set color options
nocolor = 0

# SSH timeout
timeout = 120

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o TCPKeepAlive=yes -o ServerAliveInterval=5 -o ServerAliveCountMax=3
scp_if_ssh=True
```

### OpenStack deployment using OpenStack-Ansible project scripts.

Once the network configuration of the controller and compute hosts is properly done, deploying openstack using openstack-ansible is very simple. Follow the commands below.

```
cd /opt/openstack-ansible/playbooks
openstack-ansible setup-hosts.yml
openstack-ansible haproxy-install.yml
openstack-ansible setup-infrastructure.yml
```

#### Keystone installation
Keystone is an OpenStack project that provides Identity, Token, Catalog and Policy services for use specifically by projects in the OpenStack family. It implements OpenStack’s Identity API \[[1]\].

```
openstack-ansible os-keystone-install.yml
```

#### Glance installation
The Glance project provides a service where users can upload and discover data assets that are meant to be used with other services. This currently includes images and metadata definitions.

Glance image services include discovering, registering, and retrieving virtual machine images. Glance has a RESTful API that allows querying of VM image metadata as well as retrieval of the actual image \[[2]\].

```
openstack-ansible os-glance-install.yml
```

#### Cinder installation
Cinder is a Block Storage service for OpenStack. It's designed to allow the use of either a reference implementation (LVM) to present storage resources to end users that can be consumed by the OpenStack Compute Project (Nova). The short description of Cinder is that it virtualizes pools of block storage devices and provides end users with a self service API to request and consume those resources without requiring any knowledge of where their storage is actually deployed or on what type of device \[[3]\].

```
openstack-ansible os-cinder-install.yml
```

#### Nova installation
Nova is an OpenStack project designed to provide power massively scalable, on demand, self service access to compute resources \[[4]\].

```
openstack-ansible os-nova-install.yml
```

#### Neutron installation
OpenStack Neutron is an SDN networking project focused on delivering networking-as-a-service (NaaS) in virtual compute environments. Neutron has replaced the original networking application program interface (API), called Quantum, in OpenStack. Neutron is designed to address deficiencies in “baked-in” networking technology found in cloud environments, as well as the lack of tenant control in multi-tenant environments over the network topology and addressing, which makes it hard to deploy advanced networking services \[[5]\].

```
openstack-ansible os-neutron-install.yml
```

#### Heat installation
Heat is the main project in the OpenStack Orchestration program. It implements an orchestration engine to launch multiple composite cloud applications based on templates in the form of text files that can be treated like code. A native Heat template format is evolving, but Heat also endeavours to provide compatibility with the AWS CloudFormation template format, so that many existing CloudFormation templates can be launched on OpenStack. Heat provides both an OpenStack-native ReST API and a CloudFormation-compatible Query API \[[6]\].

```
openstack-ansible os-heat-install.yml
```

#### Horizon installation
Horizon is the canonical implementation of OpenStack’s Dashboard, which provides a web based user interface to OpenStack services including Nova, Swift, Keystone, etc \[[7]\].

```
openstack-ansible os-horizon-install.yml
```

### Congratulations you have built an OpenStack Cloud
Now go ahead and access the horizon dashboard from any internet browser ```http://<Controller-IP>``` . For example

```
http://120.11.1.10
```
OpenStack is setup up successfully if horizon dashboard is accesable. Although OpenStack is up and running network is not setup yet. Different instructions are provided in the documentation for creating different network configurations.

***Documentation in progress for Network config***

<!--
#### Ceilometer installation
The Ceilometer project aims to deliver a unique point of contact for billing systems to acquire all of the measurements they need to establish customer billing, across all current OpenStack core components with work underway to support future OpenStack components \[[8]\].

```
openstack-ansible os-ceilometer-install.yml
```

#### Aodh installation
Aodh is the alarm engine of the Ceilometer project

```
openstack-ansible os-aodh-install.yml
```

#### Swift installation
Swift is a highly available, distributed, eventually consistent object/blob store. Organizations can use Swift to store lots of data efficiently, safely, and cheaply \[[9]\].

```
openstack-ansible os-swift-install.yml
```
-->


[1]: http://docs.openstack.org/developer/keystone/
[2]: http://docs.openstack.org/developer/glance/
[3]: https://wiki.openstack.org/wiki/Cinder
[4]: http://docs.openstack.org/developer/nova/
[5]: https://www.sdxcentral.com/resources/open-source/what-is-openstack-quantum-neutron/
[6]: https://wiki.openstack.org/wiki/Heat
[7]: http://docs.openstack.org/developer/horizon/
[8]: http://docs.openstack.org/developer/ceilometer/
[9]: http://docs.openstack.org/developer/swift/


Once the OSCAR setup is done, OpenStack should be up and running. The next important step would creating a virtual external network so that the VM's on top of our vDC can commounicate with outside world (Internet). For this we have pull a pool of IP's from chameleon shared net and add it to our virtual external network. Follow the steps shown below.

#### Requesting IP's from chameleon shared net

By default ```neutronclient``` should be installed on chameleon cloud nodes. If it is not installed, install it using pip

```
pip install python-neutronclient
```

From your project on Chalmeleon dashboard **download the api access** and **save** the contents to ```cc_openrc``` in the home directory of the controller node. These steps for requesting a pool of IP's from chameleon cloud can be executed from any machine, there is no restricition that this should be done from the contoller node itself. But Creating virtual external network for the OSCAR vDC should be done on chameleon cloud itself.

Source cc_openrc file and enter your chameleon account password. This file exports all credentials requeired for communicating with chameleon cloud as environmental variables and the following commands use these environment variables while talking to chameleon cloud.

```
# Make sure you read the above two paragraphs and
# downloaded the required contents before proceeding forward.
source cc_openrc
```

Check to see if chameleon cloud is reponding for your neutron-clint commands. The following command gives a list for networs available on chamleon cloud.

```
neutron net-list
```

If something similar to this is displayed then the neutron-client is working fine. If it is not please visit back the ```source cc_openrc``` step and make sure password you provided is right

```
+---------------------+------------+-------------------------------+
| id                  | name       | subnets                       |
+---------------------+------------+-------------------------------+
| <network uuid>      | sharednet1 | <sub-net uuid> 10.20.108.0/23 |
| <network uuid>      | ext-net    | <sub-net uuid>                |
+---------------------+------------+-------------------------------+
```

Now request chameleon cloud for a pool of IP's. The following step requests 3 IP's

```
neutron port-create sharednet1

neutron port-create sharednet1

neutron port-create sharednet1
```

When you type in ```neutron port-create sharednet1``` you should see an output something similar to this

```
Created a new port:
+-----------------------+---------------------------------------------------------------+
| Field                 | Value                                                         |
+-----------------------+---------------------------------------------------------------+
| admin_state_up        | True                                                          |
| allowed_address_pairs |                                                               |
| binding:vnic_type     | normal                                                        |
| device_id             |                                                               |
| device_owner          |                                                               |
| fixed_ips             | {"subnet_id": "<sub-net uuid>", "ip_address": "10.20.109.33"} |
| id                    | <some uuid>                                                   |
| mac_address           | < mac_address of the port>                                    |
| name                  |                                                               |
| network_id            | <network uuid>                                                |
| security_groups       | <security group related uuid>                                 |
| status                | DOWN                                                          |
| tenant_id             | <project name>                                                |
+-----------------------+---------------------------------------------------------------+
```

Save the IP address from these messages and we will be using these IP addresses in coming steps.

Unset the following environment variables if you are doing this process on controller because, we are about to source openrc for our OSCAR vDC and our OSCAR vDC doesn't have TENANT_ID and REGION_NAME parameters so when we source it, the commands assumes that these two are available on our vDC and gives us an error

```
unset OS_TENANT_ID
unset OS_REGION_NAME
```


#### Creating virtual external network for the OSCAR vDC

The following steps would guide you in creating an virtual external network for OSCAR vDC. **List the containers running on the controller node and attach to the neutron agents container**.

```
# List the containers
lxc-ls
```

In our case name of  **neutron agents container** was **aio1_neutron_agents_container-128a859f** so to attach to the container we used

```
lxc-attach --name aio1_neutron_agents_container-128a859f
```

Now to create an external virtual network with name **v-ext-net** use the following command

```
neutron net-create --provider:physical_network=flat --provider:network_type=flat --shared --router:external v-ext-net
```

Now use the following command to create a sub-net for **v-ext-net** with name **v-ext-sub-net** and add the allocation pool of IP's gathered in the intial steps. If the IP's gathered are in in order say ```10.xx.xx.10, 10.xx.xx.11, 10.xx.xx.12``` then in the command directly use ```--allocation-pool start=10.xx.xx.10,end=10.xx.xx.12``` if the gathered IP's are not in order then different allocation pools with same start and end IP's should be used. The command below uses different allocation pools which means the gathered IP's are not in order.

```
neutron subnet-create v-ext-net 10.20.108.0/23 --name v-ext-sub-net --gateway=10.20.109.254 --allocation-pool start=10.xx.xx.11,end=10.xx.xx.11 --allocation-pool start=10.xx.xx.13,end=10.xx.xx.13 --allocation-pool start=10.xx.xx.15,end=10.xx.xx.15
```

Finally create a router and interface it with v-ext-net

```
neutron router-create v-router

neutron router-gateway-set v-router v-ext-net
```

The virtual external network is setup. The next steps would be to setup a private network using horizon, interfacing it with ```v-router``` and spinning an instance on private network.

To create internal network

```
neutron net-create --provider:network_type=vxlan --shared internal_net
neutron subnet-create internal_net 192.168.1.0/16 --name internal_subnet --dns-nameservers list=true 8.8.8.8 
```



