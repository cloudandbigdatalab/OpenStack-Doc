# Neutron – Openstack Networking Service

### 1.	Introduction

##### 1.1 Definition

Neutron is an OpenStack project to provide "networking as a service" between interface devices (e.g., vNICs) managed by other Openstack services (e.g., nova).[1]

##### 1.2 Why Neutron

 It is increasingly becoming difficult for data centers to provide network to a platform whose network requirements keep     changing from time to time. Old traditional ways for delivering network topologies and infrastructure is no longer the norm
 of the day. Clients want instantaneous network provision on demand. To cope up with this, Openstack has launched the neutron
 service. From openstack’s perspective, main goals of neutron are: 

- Providing connectivity to virtual machines with each other and external network.
- Maintaining the isolation in communication of VMs belonging to different tenants. Keep   each tenant isolated.
- Help tenants create their own Networks, routers, complex networking topologies etc. Idea of networking virtualization.
- *aas as a service. Ex: Load Balancer as a service, Firewall as a service.


##### 1.3 Core Concepts of Neutron

 Neutron utilizes three main resources:

 - Ports: A connection point for attaching a single device NIC to neutron network.
 - Network: An isolated L2 segment analogous to VLAN. Also called as a broadcast domain.
 - Subnets: A range of V4 or V6 IP addresses associated with a configuration state.
 ![Image](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/coreconcepts.png)

 
### 2.	Architecture


There are many aspects of neutrons architecture that we need to discuss. First we will start discussing the various components of the neutron architecture. Then we shall dig deep into each component. 

  ![Image2](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/blockdiagram.png)

##### 2.1 The Block diagram
 


The above block shows the core components of neutron. Neutron Server component receives all the API calls and translates them to respective plugins for implementing the desired functionality. For traction, management and analytics, each service rendered by neutron is persisted in the neutron database. This database is a typical Maria DB installation and can be installed in the same controller node as the neutron server or can be installed on a separate node. Plugins give the neutron the capability to provide varied network services. Being pluggable with various plugins has given neutron the required potential to deliver any complicated network topologies that a tenant requires and also provides the necessary capabilities to modify these networks with least downtime. It is not always necessary that all the restful API requests received by neutron server are handled by plugins. 

The original OpenStack Compute network implementation assumed a very basic model of performing all isolation through Linux VLANs and IP tables. OpenStack Networking introduces the concept of a plug-in, which is a pluggable back-end implementation of the OpenStack Networking API. A plug-in can use a variety of technologies to implement the logical API requests. Some OpenStack Networking plug-ins might use basic Linux VLANs and IP tables, while others might use more advanced technologies, such as L2-in-L3 tunneling or OpenFlow, to provide similar benefits. [2]

Plugin agent (quantum-*-agent): Runs on each hypervisor to perform local vswitch configuration (if your neutron configuration is based on Open VSwitch technology). Agent to be run depends on which plug-in you are using, as some plug-ins do not require an agent. DHCP agent (quantum-dhcp-agent): Provides DHCP services to tenant networks. This agent is the same across all plug-ins. L3 agent (quantum-L3-agent): Provides L3/NAT forwarding to provide external network access for VMs on tenant networks. This agent is the same across all plug-ins. [2] 

Plug-ins and various internal components of neutron utilize messaging queue to communicate with each other. Most of these communications are RPC (remote procedure call) requests for fulfilling API request. 


##### 2.2 Types of networks in Openstack

 
![Image3](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/typesofnetworks.png)

Provider Network: Created by admin, mapped to pre-existing network in datacenter, used for external networks. VMs may also be directly connected to provider network. Provider network are mainly used to model or simulate all or few of the physical networking hardware.

Tenant Network: These are the networks created by neutron for the tenants on a request sent by an authorized user of that tenant. These networks are isolated from each other. These networks act as virtual private networks belonging to individual tenants and each VM belonging to that tenant is connected to that tenant network.

 Such tenant networks cannot communicate to external network, or receive communication from external network unless they somehow hook-up with one of the provider networks provided by the network administrator of that data center.


##### 2.3 Communications

As we all might have realized by now that all the components of neutron are necessarily installed on a single node. And also these components keep communicating with each other to get the job done. Communication between various Openstack components and services can be classified broadly into four types. 

 ![Image4](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/2.3%20communication.png)

Management Communication: This kind of communication takes place between various components of Openstack services in order to work together to accomplish the requested task. Most of the components may use RPC means of communication using RabbitMQ. This sort of communication uses separate network other than the Tenant or provider network we discussed earlier.

VM Data: This comprises of the communication of VMs within a tenant network. 

API Network: This network enables calling the API server requesting for implementation of various services.
Internet: Acceptable external network.

 
![Image4](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/Acceptable%20external%20network.png)


### 3.	Interaction with other Openstack Services

##### 3.1 Neutron interaction with Nova

Here we shall discuss the detailed interaction between Nova and Neutron starting from a scenario in which the Nova API server receives a request to create a new VM.

- Step1: Nova API server receives a request to create a VM.

- Step2: Nova API server transfer this call to Nova conductor which presides over the scheduling process in an attempt to      choose the compute node on which the newly created VM would reside.

- Step3: Nova schedules the bottom leftmost compute node.

- Step4: A Remote Procedure Call or RPC is invoked on the selected compute node to create a VM. This RPC request is raised by   the Nova Conductor and transferred using RabbitMQ.
- Step5: Once a VM is created, the nova compute node sends a REST API call to Neutron server to create a new port. In this     attempt, Neutron receives all the information about the newly created VM.

- Step6: At this stage the local hypervisor software or agent on the hypervisor on the compute node called “Libvirt” will      create a tap device. A tap device, in linux world, is a way of representing a virtual NIC (network interface card) in a VM.   This forms a virtual connection point where the newly created VM can be hooked onto a network.

- Step7: Following the request to create a new port, Neutron service creates a new port. It is important to recollect that a   port, in our context, means a pair of MAC and IP address. Not only does neutron creates these, it also persists them in      neutron database. Now our VM has an IP address and the virtual NIC of the VM has a MAC address.

- Step8: The neutron service sends a RPC message to the DHCP agent on the network node to notify the DHCP agent about the      newly created VM and the IP and MAC it is associated with. The DHCP agent updates its internal files so that next time it    receives request for IP address from same VM it can get it out accurately.

- Step9: A similar RPC notification is also provided to the L2 Agent residing on the same compute node the VM was created.     From now onwards this L2 agent will take over.

- Step10: In order to hookup our VM to a network, L2 agent sends an RPC message to neutron service requesting additional       details about the network devices and components. 

- Step11: Once the L2 agent is updated with the latest networking information it configures a local virtual network (VLAN)     and configures required Open VSwitch flows. Basically all L2 is doing is making sure that the tap device created earlier is   connected to proper network.

- Step12: A confirmation RPC message is sent to neutron service from L2 agent confirming the successful connection of the VM   to the right network.

- Step13: Neutron in turn communicates the same to Nova server via an API call.

- Step14: Nova server sends the same notification to Nova compute on compute node and Nova compute service boots the VM.

![Image5](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/images/3.1.png)

These steps are summarized the following slide (curtesy: Assaf Muller’s Blog [3]) :

 

##### 3.2 Neutron interaction with Keystone

<work in progress>

### 4.	Command Line Interface

##### 4.1 List the extensions for system

```sh
$ neutron ext-list -c alias -c name

+-----------------+--------------------------+
| alias           | name                     |
+-----------------+--------------------------+
| agent_scheduler | Agent Schedulers         |
| binding         | Port Binding             |
| quotas          | Quota management support |
| agent           | agent                    |
| provider        | Provider Network         |
| router          | Neutron L3 Router        |
| lbaas           | LoadBalancing service    |
| extraroute      | Neutron Extra Route      |
+-----------------+--------------------------+
```
##### 4.2 Create a network

```sh
$ neutron net-create net1

Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 2d627131-c841-4e3a-ace6-f2dd75773b6d |
| name                      | net1                                 |
| provider:network_type     | vlan                                 |
| provider:physical_network | physnet1                             |
| provider:segmentation_id  | 1001                                 |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | 3671f46ec35e4bbca6ef92ab7975e463     |
+---------------------------+--------------------------------------+
```

##### 4.3 Create a network with specified provider network
```sh
$ neutron net-create net2 --provider:network-type local

Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 524e26ea-fad4-4bb0-b504-1ad0dc770e7a |
| name                      | net2                                 |
| provider:network_type     | local                                |
| provider:physical_network |                                      |
| provider:segmentation_id  |                                      |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | 3671f46ec35e4bbca6ef92ab7975e463     |
+---------------------------+--------------------------------------+
```	

##### 4.4 Create a subnet
```sh
$ neutron subnet-create net1 192.168.2.0/24 --name subnet1

Created a new subnet:
+------------------+--------------------------------------------------+
| Field            | Value                                            |
+------------------+--------------------------------------------------+
| allocation_pools | {"start": "192.168.2.2", "end": "192.168.2.254"} |
| cidr             | 192.168.2.0/24                                   |
| dns_nameservers  |                                                  |
| enable_dhcp      | True                                             |
| gateway_ip       | 192.168.2.1                                      |
| host_routes      |                                                  |
| id               | 15a09f6c-87a5-4d14-b2cf-03d97cd4b456             |
| ip_version       | 4                                                |
| name             | subnet1                                          |
| network_id       | 2d627131-c841-4e3a-ace6-f2dd75773b6d             |
| tenant_id        | 3671f46ec35e4bbca6ef92ab7975e463                 |
+------------------+--------------------------------------------------+
```
The subnet-create command has the following positional and optional parameters:
*	The name or ID of the network to which the subnet belongs.
In this example, net1 is a positional argument that specifies the network name.
*	The CIDR of the subnet.
In this example, 192.168.2.0/24 is a positional argument that specifies the CIDR.
*	The subnet name, which is optional.
In this example, --name subnet1 specifies the name of the subnet.

##### 4.5 Create a router 
```sh
$ neutron router-create router1

Created a new router:
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| admin_state_up        | True                                 |
| external_gateway_info |                                      |
| id                    | 6e1f11ed-014b-4c16-8664-f4f615a3137a |
| name                  | router1                              |
| status                | ACTIVE                               |
| tenant_id             | 7b5970fbe7724bf9b74c245e66b92abf     |
+-----------------------+--------------------------------------+
```

Link the router to the external provider network
```sh
$ neutron router-gateway-set ROUTER NETWORK
```
Link the router to subnet
```sh
$ neutron router-interface-add ROUTER SUBNET
```
##### 4.6 Create ports
Create a port with specified IP 
```sh
$ neutron port-create net1 --fixed-ip ip_address=192.168.2.40

Created a new port:
+----------------------+----------------------------------------------------------------------+
| Field                | Value                                                                |
+----------------------+----------------------------------------------------------------------+
| admin_state_up       | True                                                                 |
| binding:capabilities | {"port_filter": false}                                               |
| binding:vif_type     | ovs                                                                  |
| device_id            |                                                                      |
| device_owner         |                                                                      |
| fixed_ips            | {"subnet_id": "15a09f6c-87a5-4d14-b2cf-03d97cd4b456", "ip_address... |
| id                   | f7a08fe4-e79e-4b67-bbb8-a5002455a493                                 |
| mac_address          | fa:16:3e:97:e0:fc                                                    |
| name                 |                                                                      |
| network_id           | 2d627131-c841-4e3a-ace6-f2dd75773b6d                                 |
| status               | DOWN                                                                 |
| tenant_id            | 3671f46ec35e4bbca6ef92ab7975e463                                     |
+----------------------+----------------------------------------------------------------------+
```
##### 4.7 Create a port without specified address
```sh
$ neutron port-create net1

Created a new port:
+----------------------+----------------------------------------------------------------------+
| Field                | Value                                                                |
+----------------------+----------------------------------------------------------------------+
| admin_state_up       | True                                                                 |
| binding:capabilities | {"port_filter": false}                                               |
| binding:vif_type     | ovs                                                                  |
| device_id            |                                                                      |
| device_owner         |                                                                      |
| fixed_ips            | {"subnet_id": "15a09f6c-87a5-4d14-b2cf-03d97cd4b456", "ip_address... |
| id                   | baf13412-2641-4183-9533-de8f5b91444c                                 |
| mac_address          | fa:16:3e:f6:ec:c7                                                    |
| name                 |                                                                      |
| network_id           | 2d627131-c841-4e3a-ace6-f2dd75773b6d                                 |
| status               | DOWN                                                                 |
| tenant_id            | 3671f46ec35e4bbca6ef92ab7975e463                                     |
+----------------------+----------------------------------------------------------------------+
```
##### 4.8 Query ports with specified fixed IP address
```sh
$ neutron port-list --fixed-ips ip_address=192.168.2.2 \
  ip_address=192.168.2.40

+----------------+------+-------------------+-------------------------------------------------+
| id             | name | mac_address       | fixed_ips                                       |
+----------------+------+-------------------+-------------------------------------------------+
| baf13412-26... |      | fa:16:3e:f6:ec:c7 | {"subnet_id"... ..."ip_address": "192.168.2.2"} |
| f7a08fe4-e7... |      | fa:16:3e:97:e0:fc | {"subnet_id"... ..."ip_address": "192.168.2.40"}|
+----------------+------+-------------------+-------------------------------------------------+
```

##### 5 Code Walk-through:
```sh
Make return values from objects db api consistent:
Previously in db api, we returned dictionaries from create() and update()
methods while get() methods returned db models. This patch makes the
return values consistent and thus we always get db model from db api
operations.

Files Modified:
 	neutron/objects/base.py
	neutron/objects/db/api.py 


base.py:
	((result = dict(db_obj)))
	result = dict(db_obj)
  	result = {field: value for field, value in dict(db_obj).items() if value is not None}

api.py:
	((return db_obj.__dict__))
	return db_obj

---------------------------------------------------------------------------------------

Remove vlan tag before injecting packets to port:

Open vSwitch takes care of vlan tagging in case normal switching is
used. When ingress traffic packets are accepted, the
actions=output:<port_number> is used but we need to explicitly take care
of stripping out the vlan tags.

Files Modified:
doc/source/devref/openvswitch_firewall.rst
neutron/agent/linux/openvswitch_firewall/firewall.py
neutron/agent/linux/openvswitch_firewall/rules.py
neutron/tests/unit/agent/linux/openvswitch_firewall/test_firewall.py
neutron/tests/unit/agent/linux/openvswitch_firewall/test_rules.py 

openvswitch_firewall.rst
((+ table=81, priority=100,arp,reg5=0x1,dl_dst=fa:16:3e:a4:22:10 actions=output:1))
+ table=81, priority=100,arp,reg5=0x1,dl_dst=fa:16:3e:a4:22:10 actions=strip_vlan,output:1

firewall.py:

{{
     def _initialize_ingress(self, port):
@@ -560,7 +561,7 @@ def _initialize_ingress(self, port):
             dl_type=constants.ETHERTYPE_ARP,
             dl_type=constants.ETHERTYPE_ARP,
             reg_port=port.ofport,
             reg_port=port.ofport,
             dl_dst=port.mac,
             dl_dst=port.mac,
-            actions='output:{:d}'.format(port.ofport),
+            actions='output:{:d}'.format(port.ofport),
         )
         )
}}

     def _initialize_ingress(self, port):
@@ -560,7 +561,7 @@ def _initialize_ingress(self, port):
             dl_type=constants.ETHERTYPE_ARP,
             dl_type=constants.ETHERTYPE_ARP,
             reg_port=port.ofport,
             reg_port=port.ofport,
             dl_dst=port.mac,
             dl_dst=port.mac,
-            actions='output:{:d}'.format(port.ofport),
+            actions='strip_vlan,output:{:d}'.format(port.ofport),
         )
         )

rules.py:
{{	flow_template['actions'] = "output:{:d}".format(port.ofport)	}}
	flow_template['actions'] = "strip_vlan,output:{:d}".format(port.ofport)

test_firewall.py:

actions='ct(commit,zone=NXM_NX_REG6[0..15]),'
-                self.port_ofport),
+            'strip_vlan,output:{:d}'.format(self.port_ofport),
             dl_dst=self.port_mac,
             dl_dst=self.port_mac,

test_rules.py:
'actions': 'strip_vlan,output:1',
```
	


