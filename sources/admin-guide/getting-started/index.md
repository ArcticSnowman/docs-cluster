# Getting Started

This document will show you how to get up and running with the Gluu Cluster Server. It is broken down into the following sections:

[TOC]

## Overview

The Gluu Cluster is divided into three separate packages, the master package,the consumer package and the cluster API.
The consumer package and the cluster API is dependent on the master package to function, therefore the master package must be installed first.
The deployment section will cover some basics of installation; for a detailed installation guide please see the [Installation Doc](../installation/)

## Deployment

### Cluster Master Package

The master package is the primary package for cluster services, but it is not stand-alone. A single master will not complete the cluster.
Run the following commands to install the master package:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-master
```

For a detailed configuration instructions, please see the [Installation Doc](../installation/).
### Cluster API

The Gluu Cluster API is contained in the `gluu-flask` package. This package will not work without the master package and these two packages must be installed in the same host.
Run the following command to install the flask package:

`apt-get install -y gluu-flask`

The installation command will install the `gluu-flask` daemon which is automatically started by init script.

### Cluster Consumer Package

The consumer package requires the master package to function and requires a commercial license. Please see the [pricing](gluu.org/pricing) for more details.

Run the following commands to install the consumer package:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-consumer
```


