[TOC]
# Installation
The Gluu Cluster only supports Ubuntu for now. There is only one mandatory package need to be installed in a machine (whether it's a remote VM, PC, or even laptop); the `gluu-flask` package.
There is one additional package, `gluu-cluster-webui`, that provides a user friendly way of using the API and managing the cluster. Note, this package must be installed in same host with `gluu-flask`.

## Prerequisites

### Minimum Linux Kernel

Cluster requires at least kernel 3.10 at minimum. We can check whether we're using supported kernel.

    uname -r

Please note, due to [issue with kernel 3.13.0-77](../known-issues#unsupported-kernel), this version should be avoided.

Also it’s recommended to install the linux-image-extra kernel package. The linux-image-extra package allows us to use the aufs storage driver for docker engine.

    apt-get update
    apt-get install linux-image-extra-$(uname -r)

## Gluu Cluster API

### Installing gluu-flask Package

First things first, we need to add Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
```

Run the following commands to install the `gluu-flask` package:

```
apt-get install -y gluu-flask
```

The command will install a daemon which is automatically started by init script. There are a few commands that are available for gluu-flask. The available commands are

	service gluu-flask start
	service gluu-flask restart
	service gluu-flask stop
	service gluu-flask status

## Gluu Cluster Web Interface package

### Installing gluu-cluster-webui

Run the following commands to install gluu-cluster-webui:
```
apt-get install -y gluu-cluster-webui
a2dissite 000-default
service apache2 restart
```

There are few things we need to know about Gluu Cluster Web UI:

1. The application is bind to 127.0.0.1 (localhost) as currently there's no protection mechanism yet.
2. The application is listening on port 8800 to avoid conflict with port used by nodes.

First things first, we need to add Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
```

Run the following commands to install the `gluu-cluster-webui` package:

```
apt-get install -y gluu-cluster-webui
```
