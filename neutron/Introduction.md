# Introduction
Neutron is Openstack's Networking service provider. Main goals of Neutron are: - <br />
<ul>
  <li>
    Providing connectivity to virtual machines with each other and external network.
  </li>
  <li>
    Maintaining the isolation in communication of VMs belonging to different tenants. i.e Keep each tenant isolated.
  </li>
  <li>  
    Help tenants create their own Networks, routers, complex networking topologies etc. i.e Idea of networking virtualization.
  </li>
  <li>
    *aas as a service. Ex: Load Balancer as a service , Firewall as a service.
  </li>
</ul>


# Architecture

<p>Neutron utilizing 3 main resources: - </p>

<ul>
  <li> Port - A connection point for attaching a single device NIC to neutron network.</li>
  <li> Network - An isolated L2 segment similar to VLAN. It can also be defined as a broadcast domain.</li>
  <li> Subnet - A block of IP addresses and associated configure state.</li>
</ul>

![Alt Text](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/neutron/netronCoreConcepts.JPG?raw=true) <br/>

Next we will discuss about various types of networks in neutron: <br/>

![alt tag](https://raw.githubusercontent.com/cloudandbigdatalab/OpenStack-Projects/master/neutron/providerNTenant.JPG) <br/>

# Installation

# CLI 

# Code Review

# Code Contribution
