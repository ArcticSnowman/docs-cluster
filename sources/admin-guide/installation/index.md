# Installation
[TOC]

The Gluu Cluster only supports Ubuntu for now. There are three packages that completes the cluster; the master, the consumer and the cluster API. The master package and the cluster API must be installed in the same host. The consumer package is not necessary for a single server installation. There is one additional package, gluu-webui, that provides a user friendly way of using the API and managing the cluster.

## Prerequisites

### Minimum Linux Kernel

Cluster requires at least kernel 3.10 at minimum. We can check whether we're using supported kernel.

    uname -r

Also it’s recommended to install the linux-image-extra kernel package. The linux-image-extra package allows us to use the aufs storage driver for docker engine.

    apt-get update
    apt-get install linux-image-extra-$(uname -r)

### Available Entropy

In some virtual machines hosted at cloud providers, oxAuth/oxTrust/oxIdp may have a slow start due to insufficient entropy.
To check available entropy, we can use the following command:

    cat /proc/sys/kernel/random/entropy_avail

If the number returned from the command is above 1000, we can __skip__ this section.
Otherwise, we need to feed the entropy with other tools.

One of the tools that we can use is `rng-tools`. It's available from Ubuntu repository:

    apt-get install rng-tools

*Note:* we probably will encounter error starting `rng-tools` service. Refer to [Troubleshooting](../troubleshooting/#unable-to-start-rng-tools-service) page for details.

After `rng-tools` service is running successfully, we can check the available entropy again.

    cat /proc/sys/kernel/random/entropy_avail

The number returned from the command above is around 3000.
From this point, we can say that we have enough entropy required
by oxAuth/oxTrust/oxIdp to start properly.

## Gluu Cluster Master

There are various components that make the cluster possible. The components are listed in the [components](../components/) page. The components are packaged into a single `deb` package to minimize the complexity of the situation. The package is named `gluu-master`.

### Installing gluu-master

First things first, we need to add Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
```

Run the following commands to install the `gluu-master` package:

```
apt-get install -y gluu-master
```

### Configuring gluu-master base package

A script called the `postinstall.py` should be downloaded and run to configure the gluu-master package.

Run the following command to download the script:

    wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py

Make a note of the IP address of the master package as it will be used in the script. Run the script using the following command:

    python postinstall.py

Enter the values asked by the prompt when the script is run, and press enter to continue. An example of the script's prompt is given below:

```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : master
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
IP address of this server: 128.199.242.74
Password for TLS certificate:
Re-type password for TLS certificate:
```

## Installing Cluster API package

The cluster API, `gluu-flask` must be installed in the same host as gluu-master.

Run the following command to install gluu-flask:

    apt-get install -y gluu-flask

The command will install a daemon which is automatically started by init script. There are a few commands that are available for gluu-flask. The available commands are

	service gluu-flask start
	service gluu-flask restart
	service gluu-flask stop
	service gluu-flask status

## Installing Gluu Agent package on Master Provider

Gluu Agent is a tool that lives on each provider. To install Gluu Agent,
type the following command:

    apt-get install -y gluu-agent

The package will start a service called `gluu-agent`.

One of Gluu Agent jobs is to recover nodes automatically after provider is rebooted.
Refer to [recovery](../recovery/index.md) page for details.

## Gluu Cluster Web Interface package

The Cluster Web Interface package, gluu-cluster-webui, offers a user interface accessible via web to manage the cluster using the cluster API.

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

## Gluu Cluster Consumer

The consumer package consists of various componenents listed in the [components page](../components/). The components are packaged into a single `deb` package called gluu-consumer.

### Installing gluu-consumer

First things first, we need to add Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
```

Run the following commands to install the `gluu-consumer` package:

```
apt-get install -y gluu-consumer
```

### Configuring gluu-consumer
A script called the `postinstall.py` should be downloaded and run to configure the gluu-consumer package.

Run the following command to download the script:

    wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py

Make a note of the IP address of the gluu-master before running the script. Run the script using the following command:

    python postinstall.py

Enter the values asked by the prompt when the script is run, and press enter to continue. An example of the script's prompt is given below:
```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : consumer
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
IP address of this server: 128.199.242.75
Password for TLS certificate:
Re-type password for TLS certificate:
```

## Installing Gluu Agent on Consumer Provider

Gluu Agent is a tool that lives on each provider. To install Gluu Agent,
type the following command:

    apt-get install -y gluu-agent

The package will start a service called `gluu-agent`.

One of Gluu Agent jobs is to recover nodes automatically after provider is rebooted.
Refer to [recovery](../recovery/index.md) page for details.
