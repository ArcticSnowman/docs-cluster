# Components

The Gluu Cluster takes advantage of some of the latest and greatest free open source components to provide a hassle-free management of the cluster system. The following components make Gluu Cluster an easy solution for the enterprise.

## Gluu Flask

Gluu-flask is a [flask](http://flask.pocoo.org/) application that publishes API's and it is combined with the Crochet project. It is  also capable of handling asynchronous events.

## Gluu Agent

Gluu Agent is a Python library consists of various commands to ensure provider is reachable within cluster. One of its notable features
is self-recovery that automatically executed after server reboot.

## Gluu Cluster UI

Gluu Cluster UI is a Python application that can be used to manage the cluster through web-based User Interface.

## Docker

[Docker](https://www.docker.com/) is a container service "that can package an application and its dependencies in a virtual container that can run on any Linux server. This helps enable flexibility and portability on where the application can run, whether on premises, public cloud, private cloud, bare metal, etc."<cite>[1]</cite>
[1]:http://www.linux.com/news/enterprise/cloud-computing/731454-docker-a-shipping-container-for-linux-code

## Salt
Salt is a configuration management system that can make sure that a set of packages are installed and running.
Salt is used to execute commands on the remote system and to copy files to docker instances. There is a salt-master and salt-minion.
To learn more about salt, please visit the [saltstack website](http://saltstack.com/)
### Salt master
Salt master is the master system in the salt setup that manages the salt minions. In short, the master has control over the minions and it can send commands and configurations to them.
Please see the [salt overview](https://docs.saltstack.com/en/getstarted/overview.html) for more information.
### Salt minion
Salt minions in the gluu-cluster are dependent on the salt master for them to function.
## Prometheus

Prometheus is a system-monitoring and alerting toolkit that is used to monitor the Gluu Cluster.
Please visit the [Prometheus Website](http://prometheus.io/) for more information.

### node-exporter

Node-exporter is used machine metric collection from the docker host. It is a third-party exporter maintained by Prometheus.

## Weave

Weave is used to connect the docker containers through a virtual network. Weave is chosen because it provides mobility in the creation of nodes as the container can be scattered across multiple hosts/cloud providers and yet the cluster will be functional. Please visit the [Weave Website](http://weave.works/) for more information.
