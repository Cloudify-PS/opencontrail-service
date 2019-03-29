# Opencontrail service
This repository contains blueprints for service creation on opencontrail integrated with openstack. 

## Overview
Purpose of this demo is to show Cloudify Manager interacting with Openstack and Opencontrail 
in order to create a service that enable traffic to flow between different openstack networks. 

The setup looks as following:
![setup](opencontrail%20demo.png)

Its divided into a two parts:
1. Provisioning - Creates networks, VMs and security group on openstack,
2. Service - Creates Network Policy on Opencontrail what enables VMs to connect.

## Prerequisites

### Openstack and Opencontrail
Opencontrail should be intergrated with openstack so the resources would be mapped from openstack to opencontrail 
and the other way. 

openstack should have a cirros image uploaded to glance in order to create VM instance from it. 
Cirros images for openstack can be obtained from following page [cirros_images](http://download.cirros-cloud.net/).

### Cloudify Manager
Cloudify Manager used in test had following properties:
- version:  4.5.5
- flavor:
    - CPU: 2
    - RAM: 4 GB
    - Disk: 40 GB
    
#### Secrets:
The deployments use secrets for holding openstack and opencontrail credentials.

Openstack:
- keystone_username
- keystone_password
- keystone_tenant_name
- keystone_url
- keystone_region

Opencontrail:
- opencontrail_user
- opencontrail_password
- opencontrail_tenant
- opencontrail_ip
- opencontrail_port
- opencontrail_domain

#### Plugins
Following plugins must be installed:
- cloudify-openstack-plugin: Tested with version 2.14.7
- cloudify-opencontrail-plugin: Tested for version 1.0.4


### Demo

#### Provisioning
First step is acheived using *provisioning.yaml* blueprint.
The blueprint creates networks (right and left), its subnets and ports and fetches a existing security group.

##### Inputs
Before creating deployment, prepare input file (*provisioning_inputs.yaml*). The following inputs must be specified:
- *security_group_name*: Name of existing security group. It must contain at least rules for passing ICMP traffic for service verification.
- *left_network_name*: Name of left network
- *left_network_subnet_name*: Name of the left subnet
- *left_subnet_cidr*: Left subnet cidr
- *right_network_name*: Name of the right network
- *right_network_subnet_name*: Name of the right subnet
- *right_subnet_cidr*: Right subnet cidr
- *image*: Name of the image used for VM instantiation. 
In the tests I used cirros image as it doesnt require private key to be assigned so password authentication from openstack console is possible.
cirros-0.4.0-x86_64 image was used for it.
- *flavor*: Openstack flavor for cirros VMs. 
CirrOS is a minimal Linux distribution so flavor can be minimal (1 CPU, 512 MB RAM and 10BG disk).

##### Installation
To install the setup using cfy, execute following command:

``cfy install provisioning.yaml -b demo_provisioning -i provisioning_inputs.yaml``

##### Verification

After installation the VMs should be available from Openstack dashboard. 
VMs are instantiated in two different networks and its impossible for them to reach each other, to verify it, open openstack console of one VM and try to ping the other.
Default credentials for cirros are cirros/gocubsgo.

#### Service
Service blueprint fetches networks created in provisioning blueprint using opencontrail plugin and creates Network Policy which enables traffic between right and left networks.

##### Inputs
Service blueprint needs only two inputs to be specified:
- *provisioning_deployment_name*: name of previously created provisioning deployment (For fetching network names)
- *network_policy_name*: Name of network policy

##### Installation  
``cfy install service.yaml -b service -i service_inputs.yaml``

##### Verification

After creation of network policy, the VMs should be able to reach each other ie. via ping.

