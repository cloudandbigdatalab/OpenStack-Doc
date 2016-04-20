##1. Introduction##

Ceilometer is a part of openstack Telemetry project and it provides Telemetry as a Service in openstack. The purpose of the openstack Telemetry project is to reliably collect data on the utilization of both physical and virtual resources and persist the collected data for subsequent retrieval and analysis to trigger actions when defined criteria are met. To achieve this Telemetry has currently following three projects:

**Aodh**: an alarming service

**Gnocchi**: a time-series database and resource indexing service

**Ceilometer**: a data collection service

Ceilometer service is used to collect the measurements on the utilization of the resources from OpenStack services and other external components, so that only one agent is needed to collect the similar data. Some examples of measurements are cpu utilization, memory utilization, instance type, bandwidth usage, power consumption etc. 

The reason why we need to collect the measurements is primarily two fold. First applications and systems require monitoring in order to ensure continuous service delivery. So we need to know whether the applications and the infrastructure running these applications have encountered any faults, and whether they are experiencing heavy utilization.  Second, in industry, it is important to be able to monitor these services to develop a method for billing customers.

Ceilometer is primarily focused on monitoring resource utilization across the cloud, although it does provide some alarming and notification functionality as well. Data collected by ceilometer can be used for capacity planning of resources, customer billing, chargeback, alarming capabilities across all OpenStack core components as well as elastic scaling.  

###1.1. Objectives###

In 2012 when ceilometer project was started there was only a single objective to provide the infrastructure that will  collect any information regarding OpenStack projects, so that rating engines can use this single infrastructure as a source to transform the collected events into billable items which is labeled as “metering”. Later another objective was added to ceilometer, it is the way to collect meter, regardless of the purpose of the collection. For example, Ceilometer can now publish information for monitoring, debugging and graphing tools in addition or in parallel to the metering backend and this is labeled as “multi-publisher”.

###1.2. Billing: A Three step Process###
**Metering**: collect resource utilization data from different services

**Rating**: Transform collected usage data into billable items 

**Billing**: create invoice, based on the collected resource utilization data and rating

##2. Ceilometer System Architecture##

The major components of the Ceilometer architecture are the polling agents, notification agents,  collectors for storing agent results, alarm evaluators, alarm notifiers, and several different backend databases to store the measurements.

Below are the different components in ceilometer architecture [2]:

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig1%20Ceilometer%20Architecture.png)
*Figure 1: Ceilometer Architecture*

###2.1. Components of Ceilometer###

There are two types of Ceilometer agents: polling agents  and notification agents.

**Polling Agents**: The polling agents periodically request various metrics from different openstack services. For   example, the ceilometer-compute agent will run on a compute node and collects metrics related to compute node like cpu utilization. It is a daemon designed to poll OpenStack services and build Meters.

**Notification Agents**: Notification agent's role is to listen on the message bus, and gather information about the inner workings of other OpenStack systems based on their notification outputs. It is a daemon designed to listen to notifications on message queue, convert them to Events and Samples, and apply pipeline actions.

**Collectors**: The data collected by the agents is sent to the ceilometer-collectors, which is a daemon that transforms the collected data and record events and metering data into the backend databases. There may be several different databases used, based upon the different types of data. 

**API**: Used to query and view data recorded by the collector in internal full-fidelity DB (if enabled).

**Alarm Evaluators**: The Alarm Evaluator is configured to look at the data in the system and evaluate whether alarming criteria are met. These criteria need to defined and manually configurable. 

**Alarm Notifiers**: Once the defined criteria is met, then Alarm Notifier will take an action based upon the raised alarm. This could be calling a specific URL, or another user defined action.

Data Collected by ceilometer can be used by other services. For example Gnocchi and Aodh. As ceilometer has grown to capture more data, it became apparent that data storage would need to be optimized. This database optimization is handled by Gnocchi. Gnocchi is intended to replace the existing metering database interface [2]. Similarly Aodh, an alarming service was introduced and making ceilometer more modular and efficient for maintaining.

###2.2. Ceilometer Architecture with Billing###

Below is the Ceilometer architectural diagram with billing and Rating System: 

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig2%20Ceilometers%20Architecture%20with%20Billing%20and%20Rating%20System.png)
*Figure 2: Ceilometers Architecture with Billing and Rating System[4]*

The above image explains how billing and rating system uses the data collected by ceilometer for generating customer invoice. Billing and Rating system will interact with Ceilometer through a RESTful API call and gets the data related to resource utilization and service utilizations across all the openstack services. Based on this data customer invoice is generated by billing and rating system. 

In the above architecture, a compute node agent runs on each compute node and polls for resource utilization data and a central agent runs on a central management server to poll for the resources utilization statistics which are not tied to instances or compute nodes. It also contains a collector, which runs on one or more central management servers to monitor the message queues. Notification messages are processed and turned into metering messages and sent back out onto the message bus and then metering measurements are written to the data store.

###2.3. Ceilometer Work Flow###

* Collect data on resource utilizations from OpenStack components 
* Transform the collected metering data 
* Publish meters to any destination (including Ceilometer itself) 
* Store received meters into database
* Aggregate samples via a REST API

##3. How Ceilometer works##

In Ceilometer, polling agents will be running across every compute node and also actively listening to notifications on the central notifications message bus where a central server collects the raw metering metrics. In addition, the central server collects usage and utilization metrics from other resources that are not tied to a specific node. This central server acts as a collector which monitors the queues of messages that are sent by the agents and buses. After the messages are processed, they are then stored in a dedicated database server that supports intensive write operations.

###3.1. Data Collection###

Below is the image that describes how data is collected in ceilometer using collectors and agents:

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig3%20Data%20Collection%20in%20Ceilometer.png)
*Figure 3: Data Collection in Ceilometer[5]*

Each and every project that you want to collect the data should send events on the Oslo bus about resource utilization. But not all projects have implemented this and you will often need to instrument other tools which may not use the same bus as OpenStack has defined. 

The Ceilometer project created 2 methods to collect data:

**Bus listener agent (Notification Agents)**: It takes events generated on the notification bus and transforms them into Ceilometer samples. This is the preferred method of data collection . Here notification daemon (agent-notification) is the main component, which monitors the message bus for data being provided by other OpenStack components such as Nova, Glance, Cinder, Neutron, Swift, Keystone, and Heat, as well as Ceilometer internal communication [5]. 

**Polling agents**: This is the less preferred method, will poll some API or other tool to collect information at a regular interval. Where the option exists to gather the same data by consuming notifications, then the polling approach is less preferred due to the load it can impose on the API services. The polling agent daemon is configured to run one or more pollster plugins using either the ceilometer.poll.compute and/ or ceilometer.poll.compute namespaces [5].

The first method is supported by the ceilometer-notification agent, which monitors the message queues for notifications. Polling agents can be configured either to poll local hypervisor or remote APIs[5].

###3.2. Processing the data###

The collected data is transformed according to the requirements of the publisher and published. This processing of data is performed by pipeline.

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig4%20Processing%20of%20data%20using%20pipeline%20.png)
*Figure 4: Processing of data using pipeline [5]*

###3.3. Publishing the data###

Processed data can be published using:

* Notifier, a notification based publisher which pushes samples to a message queue which can be consumed by the collector or an external system. 
* UDP, which publishes samples using UDP packets.
* Kafka, which publishes data to a Kafka message queue to be consumed by any system that supports Kafka.

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig5%20publishing%20the%20processed%20data.png)
*Figure 5: publishing the processed data [5]*

###3.4. Storing the data###

The collector daemon gathers the processed event and metering data captured by the notification and polling agents. It validates the incoming data and (if the signature is valid) then writes the messages to a declared target: database, file, or http [5]. Currently default database for storing in ceilometer is Mongodb. It also supports SQL databases and HBase databases. 

###3.5. Accessing the data###

Data can be directly accessed by querying the underlying databases but schemas may change . So this is not the preferred method for accessing data. Better way is to access via RESTful API calls. 

![alt tag](https://github.com/cloudandbigdatalab/OpenStack-Projects/blob/master/ceilometer/images/Fig6%20Accessing%20data%20by%20external%20systems.png)
*Figure 6: Accessing data by external systems [5]*

##4. Installation##

###4.1. Deploying Ceilometer over Devstack###

By default, ceilometer is not enabled in the devstack environment, so we need to follow below steps to configure it before we use it:

1. Download Devstack
2. Modify the local.conf file as your input to devstack.
 1. Ceilometer will use RabbitMQ
 2. An important note: The ceilometer services are not enabled by default, so they must be enabled in local.conf before running stack.sh.

	This example local.conf file shows all of the settings required for ceilometer:
	```
[[local|localrc]]
# Enable the Ceilometer devstack plugin
enable_plugin ceilometer https://git.openstack.org/openstack/ceilometer.git
	```
3. Nova does not generate the periodic notifications for all known instances by default. To enable these auditing events, set instance_usage_audit to true in the nova configuration file and restart the service.
4. Cinder does not generate notifications by default. To enable these auditing events, set the following in the cinder configuration file and restart the service:
    
###4.2 Installing Manually###
Here we describe how to install and configure the ceilometer on the controller node and the compute nodes using MongoDb. We can install using mySql also.
###4.2.1 Controller Node###
#####Change to super user mode:
```
sudo su
```
#####Install the MongoDB:
```
apt-get install -y mongodb-server mongodb-clients python-pymongo
```
#####Edit the /etc/mongodb.conf file:
```
vi /etc/mongodb.conf
bind_ip = 10.0.0.11
smallfiles = true
```
#####Restart the MongoDB service:
```
service mongodb stop
rm -f /var/lib/mongodb/journal/prealloc.*
service mongodb start
```
#####Create the ceilometer database:
```
mongo --host controller --eval '
db = db.getSiblingDB("ceilometer");
db.addUser({user: "ceilometer",
pwd: "CEILOMETER_DBPASS",
roles: [ "readWrite", "dbAdmin" ]})'
```
#####Configure service user and role:
```
vi admin_creds
#Paste the following:
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=admin_pass
export OS_AUTH_URL=http://controller:35357/v2.0

source admin_creds
keystone user-create --name ceilometer --pass service_pass
keystone user-role-add --user ceilometer --tenant service --role admin
```
#####Register the service and create the endpoint:
```
keystone service-create --name ceilometer --type metering --description "Telemetry"

keystone endpoint-create \
--service-id $(keystone service-list | awk '/ metering / {print $2}') \
--publicurl http://controller:8777 \
--internalurl http://controller:8777 \
--adminurl http://controller:8777 \
--region regionOne
```

#####Install ceilometer packages:
```
apt-get install -y ceilometer-api ceilometer-collector ceilometer-agent-central \
ceilometer-agent-notification ceilometer-alarm-evaluator ceilometer-alarm-notifier python-ceilometerclient
```
#####Edit the /etc/ceilometer/ceilometer.conf file:
```
vi /etc/ceilometer/ceilometer.conf

[database]
replace connection=sqlite:////var/lib/ceilometer/ceilometer.sqlite with:
connection = mongodb://ceilometer:CEILOMETER_DBPASS@controller:27017/ceilometer

[DEFAULT]
verbose = True

rpc_backend = rabbit
rabbit_host = controller
rabbit_password = service_pass

auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000/v2.0
identity_uri = http://controller:35357
admin_tenant_name = service
admin_user = ceilometer
admin_password = service_pass

[service_credentials]
os_auth_url = http://controller:5000/v2.0
os_username = ceilometer
os_tenant_name = service
os_password = service_pass

[publisher]
metering_secret = METERING_SECRET
```
#Restart the Telemetry services:
```
service ceilometer-agent-central restart
service ceilometer-agent-notification restart
service ceilometer-api restart
service ceilometer-collector restart
service ceilometer-alarm-evaluator restart
service ceilometer-alarm-notifier restart
```
#####Configure the Image Service for Telemetry. Edit the /etc/glance/glance-api.conf file:
```
vi /etc/glance/glance-api.conf
[DEFAULT]
notification_driver = messaging
rpc_backend = rabbit
rabbit_host = controller
rabbit_password = service_pass
```
#####Edit the /etc/glance/glance-registry.conf file:
```
vi /etc/glance/glance-registry.conf
[DEFAULT]
notification_driver = messaging
rpc_backend = rabbit
rabbit_host = controller
rabbit_password = service_pass
```
#####Restart the Image Service:
```
service glance-registry restart
service glance-api restart
```
###4.2.2 Compute Node###
#####Change to super user mode:
```
sudo su
```
#####Install the ceilometer agent package:
```
apt-get install -y ceilometer-agent-compute
```
#####Edit the /etc/nova/nova.conf:
```
vi /etc/nova/nova.conf
[DEFAULT]
instance_usage_audit = True
instance_usage_audit_period = hour
notify_on_state_change = vm_and_task_state
notification_driver = nova.openstack.common.notifier.rpc_notifier
notification_driver = ceilometer.compute.nova_notifier
```
#####Restart the Compute service:
```
service nova-compute restart
```
#####Edit /etc/ceilometer/ceilometer.conf file:
```
vi /etc/ceilometer/ceilometer.conf

[DEFAULT]
verbose = True

rabbit_host = controller
rabbit_password = service_pass

[keystone_authtoken]
auth_uri = http://controller:5000/v2.0
identity_uri = http://controller:35357
admin_tenant_name = service
admin_user = ceilometer
admin_password = service_pass

[service_credentials]
os_auth_url = http://controller:5000/v2.0
os_username = ceilometer
os_tenant_name = service
os_password = service_pass
os_endpoint_type = internalURL
os_region_name = regionOne

[publisher]
metering_secret = METERING_SECRET
```
#####Restart the Telemetry service:
```
service ceilometer-agent-compute restart
```

##5. Using CLI##

```
    alarm-combination-create    Create a new alarm based on state of other
                                alarms.
    alarm-combination-update    Update an existing alarm based on state of
                                other alarms.
    alarm-create                Create a new alarm (Deprecated). Use alarm-
                                threshold-create instead.
    alarm-delete                Delete an alarm.
    alarm-event-create          Create a new alarm based on events.
    alarm-event-update          Update an existing alarm based on events.
    alarm-gnocchi-aggregation-by-metrics-threshold-create
                                Create a new alarm based on computed
                                statistics.
    alarm-gnocchi-aggregation-by-metrics-threshold-update
                                Update an existing alarm based on computed
                                statistics.
    alarm-gnocchi-aggregation-by-resources-threshold-create
                                Create a new alarm based on computed
                                statistics.
    alarm-gnocchi-aggregation-by-resources-threshold-update
                                Update an existing alarm based on computed
                                statistics.
    alarm-gnocchi-resources-threshold-create
                                Create a new alarm based on computed
                                statistics.
    alarm-gnocchi-resources-threshold-update
                                Update an existing alarm based on computed
                                statistics.
    alarm-history               Display the change history of an alarm.
    alarm-list                  List the user's alarms.
    alarm-show                  Show an alarm.
    alarm-state-get             Get the state of an alarm.
    alarm-state-set             Set the state of an alarm.
    alarm-threshold-create      Create a new alarm based on computed
                                statistics.
    alarm-threshold-update      Update an existing alarm based on computed
                                statistics.
    alarm-update                Update an existing alarm (Deprecated).
    capabilities                Print Ceilometer capabilities.
    event-list                  List events.
    event-show                  Show a particular event.
    event-type-list             List event types.
    meter-list                  List the user's meters.
    query-alarm-history         Query Alarm History.
    query-alarms                Query Alarms.
    query-samples               Query samples.
    resource-list               List the resources.
    resource-show               Show the resource.
    sample-create               Create a sample.
    sample-create-list          Create a sample list.
    sample-list                 List the samples (return OldSample objects if
                                -m/--meter is set).
    sample-show                 Show a sample.
    statistics                  List the statistics for a meter.
    trait-description-list      List trait info for an event type.
    trait-list                  List all traits with name <trait_name> for
                                Event Type <event_type>.
    bash-completion             Prints all of the commands and options to
                                stdout.
    help                        Display help about this program or one of its
                                subcommands.
```

**Example of meter-list:**

```
   $ ceilometer meter-list
   +----------------------------+------------+-----------+---------------+-----------+--------------+
   | Name                       | Type       | Unit      | Resource ID   | User ID   | Project ID   |
   +----------------------------+------------+-----------+---------------+-----------+--------------+
   | cpu                        | cumulative | ns        | INSTANCE_ID_1 | USER_ID_A | PROJECT_ID_X |
   | cpu                        | cumulative | ns        | INSTANCE_ID_2 | USER_ID_B | PROJECT_ID_Y |
   | cpu                        | cumulative | ns        | INSTANCE_ID_3 | USER_ID_C | PROJECT_ID_Z |
   | cpu_util                   | gauge      | %         | INSTANCE_ID_1 | USER_ID_A | PROJECT_ID_X |
   | cpu_util                   | gauge      | %         | INSTANCE_ID_3 | USER_ID_C | PROJECT_ID_Z |
   | disk.ephemeral.size        | gauge      | GB        | INSTANCE_ID_1 | USER_ID_A | PROJECT_ID_X |
   | disk.ephemeral.size        | gauge      | GB        | INSTANCE_ID_2 | USER_ID_B | PROJECT_ID_Y |
   | disk.ephemeral.size        | gauge      | GB        | INSTANCE_ID_3 | USER_ID_C | PROJECT_ID_Z |
   | ... [snip]                                                                                     |
   +----------------------------+------------+-----------+---------------+-----------+--------------+
```

* Using Aggregate Statistics:
```
   $ ceilometer statistics --meter cpu_util
   +--------+--------------+------------+-------+------+-----+-----+-----+----------+----------------+----
   | Period | Period Start | Period End | Count | Min  | Max | Sum | Avg | Duration | Duration Start | ...
   +--------+--------------+------------+-------+------+-----+-----+-----+----------+----------------+----
   | 0      | PERIOD_START | PERIOD_END | 2024  | 0.25 | 6.2 | 550 | 2.9 | 85196.0  | DURATION_START | ...
   +--------+--------------+------------+-------+------+-----+-----+-----+----------+----------------+----
```
* Individual data points for a particular meter may be aggregated into consolidated statistics via the CLI statistics command:
```
   $ ceilometer sample-list --meter cpu -q 'resource_id=INSTANCE_ID_1;timestamp>2013-10-01T09:00:00;timestamp<=2013-10-01T09:30:00'
   +---------------+------+------------+---------------+------+---------------------+
   | Resource ID   | Name | Type       | Volume        | Unit | Timestamp           |
   +---------------+------+------------+---------------+------+---------------------+
   | INSTANCE_ID_1 | cpu  | cumulative | 1.7234e+11    | ns   | 2013-10-01T09:08:28 |
   | INSTANCE_ID_1 | cpu  | cumulative | 1.743e+11     | ns   | 2013-10-01T09:18:28 |
   | INSTANCE_ID_1 | cpu  | cumulative | 1.7626e+11    | ns   | 2013-10-01T09:28:28 |
   +---------------+------+------------+---------------+------+---------------------+
```


##6. References:##

[1] most of the content is taken after reading  http://docs.openstack.org/developer/ceilometer/

[2] http://docs.openstack.org/developer/ceilometer/_images/ceilo-gnocchi-arch.png

[3] http://www.slideshare.net/NicolasBarcet/ceilometer-heatequalsalarming-icehousesummit

[4]http://image.slidesharecdn.com/oscon132-130528180211-phpapp02/95/metering-and-billing-for-cloud-services-9-638.jpg?cb=1369995171

[5] http://docs.openstack.org/developer/ceilometer/_images/1-agents.png

[6] http://docs.openstack.org/developer/ceilometer/architecture.html

[7] https://www.rdoproject.org/install/ceilometerquickstart/

[8] http://docs.openstack.org/developer/ceilometer/install/manual.html

[9] https://github.com/MarouenMechtri/Ceilometer-Installation-for-OpenStack-Juno#id3

