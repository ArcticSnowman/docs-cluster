# Recovery Overview
[TOC]

A provider may crashes due to various reasons (i.e. power outage).
When it crashes, all containers (weave, prometheus, and nodes) will crash as well.

As of v0.3.0, there is a recovery script to recover all containers.
This script will check all stopped containers used by Cluster and try to re-run them.

What we need to know before running recovery script:

## Ensure API is Up and Running

```
$ curl -i -X HEAD http://localhost:8080/clusters
HTTP/1.0 200 OK
Content-Type: application/json
Server: Werkzeug/0.10.4 Python/2.7.9

curl: (18) transfer closed with 854 bytes remaining to read
```

If we get status code 200, then we're good to go to next step.

##  Get `salt-master` IP address

Typically this will be the IP used by eth0 interface of the VM where Gluu-master is installed.

```
$ ifconfig eth0
eth0    Link encap:Ethernet  HWaddr f0:79:59:86:3f:c4
        inet addr:192.168.1.3  Bcast:192.168.1.255  Mask:255.255.255.0
```

On the example above, the IP address is set to `192.168.1.3`.

```
export MASTER_IPADDR=192.168.1.3
```

## Get the Provider ID
There are two methods of getting the provider ID. There is the option of using `curl` command or using the web interface.

### Provider ID using curl 
The `curl` method to get the provider ID:

```
$ curl http://localhost:8080/providers
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "docker_base_url": "128.199.198.172:2375",
        "hostname": "master-host",
        "id": "283bfa41-2121-4433-9741-875004518677",
        "type": "master"
        "ssl_key": "multi-line contents of TLS certificate key",
        "ssl_cert": "multi-line contents of TLS certificate",
        "ca_cert": "multi-line contents of CA certificate"
    },
    {
        "docker_base_url": "128.199.198.173:2375",
        "hostname": "consumer-host",
        "id": "283bfa41-2121-4433-9741-875004518678",
        "type": "consumer"
        "ssl_key": "multi-line contents of TLS certificate key",
        "ssl_cert": "multi-line contents of TLS certificate",
        "ca_cert": "multi-line contents of CA certificate"
    }
]
```

We will try to recover consumer provider with ID `283bfa41-2121-4433-9741-875004518678`.

```
export PROVIDER_ID=283bfa41-2121-4433-9741-875004518678
```

### Provider ID using web interface
[Log into the web interface](http://www.gluu.org/docs-cluster/admin-guide/webui/#accessing-the-interface) for the cluster.

Click on `providers`.
![Provider](https://raw.githubusercontent.com/GluuFederation/docs-cluster/master/sources/img/webui_provider_menu.png)

This will bring up the following screen where from click on the specific provider for `provider ID`.
![webui_provider](https://raw.githubusercontent.com/GluuFederation/docs-cluster/master/sources/img/webui_provider.png)

Clicking on the provider will bring up its information from which the provider ID can be easily noted.
![webui_provider-id](https://raw.githubusercontent.com/GluuFederation/docs-cluster/master/sources/img/webui_provider-id.png)

## Run the Recovery Script

As we have gathered all required data about our provider, let's try to recover the provider.

    API_ENV=prod SALT_MASTER_IPADDR=$MASTER_IPADDR gluuapi recover $PROVIDER_ID

The process may take a while, depends on how many nodes were deployed on the provider.

By default, `gluu-flask` will have to wait for 30 seconds for each stopped node to ensure
inter-node communication is established.
That means, if we have 4 stopped nodes, the recovery process may take 120 seconds to finish.
We can adjust the delay time (in seconds) by passing the `--exec-delay` with our desired value.

    API_ENV=prod SALT_MASTER_IPADDR=$MASTER_IPADDR gluuapi recover $PROVIDER_ID --exec-delay 20

There's no exact formula of how long the delay time should be set. Hence experimenting with different
values is our best approach.
