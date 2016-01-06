# Getting Started

This document will show you how to get up and running with the Gluu Cluster Server. It is broken down into the following sections:

[TOC]

## Overview

The Gluu Cluster is divided into three separate packages, the master package,the consumer package and the cluster API.
The consumer package and the cluster API is dependent on the master package to function, therefore the master package must be installed first.
The deployment section will cover some basics of installation; for a detailed installation guide please see the [Installation Doc](../installation/)

## Preparing VM
Cluster installation requires some ports to be accessible by the services and components required.

### External Ports
External ports can alse be called 'internet-facing' ports that are open to the world or internet.


|	Port Protocol	|	Port Number	|	Service		|
|-----------------------|-----------------------|-----------------------|
|	TCP		|	80		|	Web Frontend	|
|	TCP		|	443		|	Web Frontend	|


*Note: These ports only need to be open when you are downloading new packages. Most often your upgrade process will be tightly controlled, so you can plan for these changes and re-open the ports as needed.

### Internal Ports
Internal ports are the specific port requirements for the different components of the Cluster setup.


|	Port Protocol	|	Port Number	|	Service		|
|-----------------------|-----------------------|-----------------------|
|	TCP		|	2375		|	Docker Daemon	|
|	TCP		|	4506 & 4505	|	Salt		|
|	TCP		|	8800\*		|	Gluu-webui	|
|	TCP		|	8080\*		|	Gluu-flask	|
|	TCP		|	8443\*		|	oxTrust GUI	|
|	TCP		|	9090\*		|	Prometheus	|
|	TCP & UDP	|	6783		|	Weave		|
|	TCP & UDP	|	53		|	Weave DNS	|

\* only needed by master provider (VM)


## Deployment

### Cluster Master Package

The master package is the primary package for cluster services, but it is not stand-alone. A single master will not complete the cluster.
Run the following commands to install the master package:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-master gluu-agent

For Centos 7
curl http://repo.gluu.org/centos/Gluu-7.repo -o /etc/yum.repos.d/Gluu-7.repo
curl http://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -o /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
yum clean all
yum install -y gluu-master gluu-agent
```

For a detailed configuration instructions, please see the [Installation Doc](../installation/).
### Cluster API

The Gluu Cluster API is contained in the `gluu-flask` package. This package will not work without the master package and these two packages must be installed in the same host.
Run the following command to install the flask package:

    apt-get install -y gluu-flask
    
    Centos 7
    yum install -y gluu-flask

The installation command will install the `gluu-flask` daemon which is automatically started by init script.

### Cluster Consumer Package

The consumer package requires the master package to function and requires a commercial license. Please see the [pricing](http://www.gluu.org/gluu-server/pricing/) for more details.

Run the following commands to install the consumer package:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-consumer gluu-agent

For Centos 7
curl http://repo.gluu.org/centos/Gluu-7.repo -o /etc/yum.repos.d/Gluu-7.repo
curl http://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -o /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
yum clean all
yum install -y gluu-consumer gluu-agent
```
