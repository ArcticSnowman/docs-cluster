# Cluster Management
[TOC]

The cluster packages consisting of `gluu-master`, `gluu-agent`, `gluu-flask` and `gluu-consumer` must be installed before configuring the cluster environment. Please see the [Installation Instructions](../installation/) for more details.

## Overview

A [Cluster](../../reference/api/cluster/) is a set of nodes deployed in one or more [Providers](../../reference/api/provider/).
The cluster contains information shared across providers, like hostname.

To manage cluster, we can use Cluster Web UI or using API directly.
Note, this page only covers how to manage cluster by using the API directly via `curl` command.
To use Web UI, refer to [Web Interface](../webui) page for details.

## Creating Cluster

The following command creates a cluster using `curl`.

```
curl http://localhost:8080/clusters \
    -d name=cluster1 \
    -d org_name=my-org \
    -d org_short_name=my-org \
    -d city=Austin \
    -d state=TX \
    -d country_code=US \
    -d admin_email='info@example.com' \
    -d ox_cluster_hostname=gluu.example.com \
    -d weave_ip_network=10.2.1.0/24 \
    -d admin_pw=secret \
    -X POST -i
```
The parameters of the command are explained below:

* `name` represents the cluster name or label with which the cluster is identified.
* `org_name`, `org_short_name`, `city`, `state`, `country_code`, and `admin_email` are used for X509 certificate.
* `ox_cluster_hostname` is a URL for the appliance; this name must be reachable from the browser.
* `weave_ip_network` is IP address network used for inter-container communication; every cluster should have an unique IP address.
* `admin_pw` is used for LDAP password, LDAP replication password, and oxTrust admin password.

A successful request returns a HTTP 201 status code:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/clusters/1279de28-b6d0-4052-bd0c-cc46a6fd5f9f

{
    "inum_org": "@!FDF8.652A.6EFF.F5A3!0001!DA7B.9EB2",
    "oxauth_nodes": [],
    "inum_appliance": "@!FDF8.652A.6EFF.F5A3!0002!FA83.4368",
    "admin_email": "info@example.com",
    "inum_appliance_fn": "FDF8652A6EFFF5A30002FA834368",
    "description": null,
    "city": "Austin",
    "base_inum": "@!FDF8.652A.6EFF.F5A3",
    "inum_org_fn": "FDF8652A6EFFF5A30001DA7B9EB2",
    "weave_ip_network": "10.2.1.0/24",
    "ldaps_port": "1636",
    "ox_cluster_hostname": "gluu.example.com",
    "state": "TX",
    "country_code": "US",
    "ldap_nodes": [],
    "nginx_nodes": [],
    "org_short_name": "my-org",
    "org_name": "my-org",
    "id": "1279de28-b6d0-4052-bd0c-cc46a6fd5f9f",
    "oxtrust_nodes": [],
    "name": "cluster1",
    "oxidp_nodes": []
}
```

We'll need the `cluster_id` when deploying nodes, so let's keep the reference to `cluster_id` as environment variable.

```
export CLUSTER_ID=1279de28-b6d0-4052-bd0c-cc46a6fd5f9f
```

## Managing Master Provider and Its Nodes

### Registering Master Provider
A provider must be registered after creating the cluster.
This creates the entity in the `gluu-flask` JSON database to let the  API's know where to deploy the instances.
The `postinstall.py` script generates TLS certificates which are stored in `/etc/docker` directory.

There are few things we need to check before registering the provider.

1.  Grab the actual server hostname. We can use `hostname -f` command. In this example, we have `gluu-master` as the server hostname.
2.  Since Gluu Cluster relies on salt, we need to double check whether salt uses the same hostname as seen in step 1.
    We can use `cat /etc/salt/minion_id` command. If the command returns `gluu-master` as well, we are safe to proceed to provider registration.
    If it's not, use value from `/etc/salt/minion_id` as provider hostname.

The following command registers a provider using `curl`.

```
curl http://localhost:8080/providers \
    -d hostname=gluu-master \
    -d docker_base_url='https://128.199.242.74:2376' \
    -d type='master' \
    -X POST -i
```

The parameters of the command are explained below:

* `hostname` is the FQDN hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-master` package. It is recommended to use `https` scheme.
* `type` is the provider type (either `master` or `consumer`)

A successful request returns a HTTP 201 status code:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/58848b94-0671-48bc-9c94-04b0351886f0

{
    "docker_base_url": "https://128.199.242.74:2376",
    "hostname": "gluu-master",
    "id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "type": "master"
}
```

The `provider_id` is required to deploy nodes, so it is best to keep the reference as an environment variable.

    export MASTER_PROVIDER_ID=58848b94-0671-48bc-9c94-04b0351886f0

A successful creation of provider follows a background job to complete internal IP routing through `weave` and importing necessary docker certificates.
By default, it may take up-to 25 seconds to finish the provider creation.

#### Imported docker Certificates

To deploy/delete nodes, `gluu-flask` uses docker Remote API protected by TLS, hence `gluu-flask` require necessary docker certificates.
These certificates are copied from host-level docker config directory (`/etc/docker`):

1. `/etc/docker/ca.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__ca.pem`
2. `/etc/docker/key.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__key.pem`
3. `/etc/docker/cert.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__cert.pem`

##### Weave Routing

Run the following command to check if routing is ready:

```sh
weave ps
```
If the routing is ready then the output will be similar to the following:
```
weave:expose ca:68:e4:e8:ed:26 10.2.1.254/24
1d34079275fc aa:d3:17:16:52:40 10.2.1.253/24
e5d1dd8d9ad4 32:7b:97:97:02:c5
```
Please use the following command to check whether `weave` in master provider ready to accept connections from other providers:

```
$ weave status
```

The command will provide a similar output as below:

```sh
$ weave status

       Version: v1.1.0

       Service: router
      Protocol: weave 1..2
          Name: d6:5f:eb:ea:b7:03(gluu-master)
    Encryption: enabled
 PeerDiscovery: enabled
       Targets: 0
   Connections: 0
         Peers: 1

       Service: ipam
     Consensus: achieved
         Range: [10.2.1.0-10.2.2.0)
 DefaultSubnet: 10.2.1.0/24

       Service: dns
        Domain: gluu.local.
           TTL: 1
       Entries: 0

```

### Deploying Nodes
The deployment of nodes can be done after the creation of Cluster and Provider entities. The nodes are interdependent, it's recommended to deploy them in the following order

1. `ldap` LDAP Node

2. `oxauth` oxAuth Node

3. `nginx` nginx Node

4. optional `oxidp` oxIDP Node

5. `oxtrust` oxTrust Node

#### LDAP Node

Run the following command to deploy the ldap node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request returns a HTTP 202 status code.
Here's an example of response returned from request above:

```
{
    "cluster_id": "1279de28-b6d0-4052-bd0c-cc46a6fd5f9f",
    "domain_name": "",
    "id": "",
    "ip": "",
    "name": "gluuopendj_51b8b62c-283e-4de8-aa64-4b112e3e036e",
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "state": "IN_PROGRESS",
    "type": "ldap",
    "weave_ip": "10.2.1.1"
}
```

Note the node's `name` from the response above.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

This technique can be applied to all nodes in the cluster.

##### Custom LDAP Schema

Starting from v0.4.0, ldap node has support for custom schema. To deploy custom schema,
put the desired schema in `.ldif` file on the same server running gluu-flask under `/var/lib/gluu-cluster/custom/opendj/schema/`.
For example, we can create `/var/lib/gluu-cluster/custom/opendj/schema/102-customSchema.ldif` for our custom schema.
This file will be added to ldap node located at `/opt/opendj/config/schema/102-customSchema.ldif`. The schema is copied on ldap server
creation.

#### oxAuth Node
Run the following command to deploy oxAuth node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxauth \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

#### nginx Node

Starting from v0.4.2, we can use signed SSL certificates instead of self-signed certificates (the ones that generated during nginx node deployment).
Assuming we have `domain.crt` and `domain.key` certificate/key files, we need to copy them to predefined paths in master provider.

1. Copy `domain.crt` to `/var/lib/gluu-cluster/ssl_certs/nginx.crt` (create the `/var/lib/gluu-cluster/ssl_certs` directory if not exist).
2. Copy `domain.key` to `/var/lib/gluu-cluster/ssl_certs/nginx.key`.

Run the following command to deploy the nginx node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=nginx \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

#### oxIdp Node (optional)
Run the following command to deploy oxIdp node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxidp \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

To generate default IdP and SP metadata, we need to deploy oxtrust node after deploying oxidp node.

#### oxTrust Node
Run the following command to deploy oxTrust node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxtrust \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

##### Accessing oxTrust Web Interface

The Gluu Server web interface can be accessed after the deployment of the nodes are complete. The oxTrust UI is run at `https://localhost:8443` in the master provider. To access the UI, `ssh` tunneling must be used. The exaple will use `128.199.242.74` as an example IP address of the master provider.

```
ssh -L 8443:localhost:8443 root@128.199.242.74
```

After the connection to 128.199.242.74:8443 is established, visit `https://localhost:8443` in the web browser.
The oxTrust login page will load asking for a username and a password.

* username: `admin`
* password: `admin_pw` value from the LDAP node

If the credentials are supplied correctly, the page will be redirected to the oxTrust dashboard.

## Managing Consumer Provider and Its Nodes

One of the purposes of creating consumer provider is to achieve HA setup in the cluster.
It means when master provider (and its nodes) is unavailable, we still have another provider (and its nodes) to serve requests.
Although creating consumer is optional step, it's recommended to do so if we want to achieve full-featured Gluu Cluster.

### Registering License Key

Registering license key is required before registering any consumer provider.
Refer to [License](../overview/#license) section to see available license types.

To register license key, we need to obtain code, public password, public key, and license password
from Gluu. Afterwards, we can store them as Gluu Cluster's license key.

The following command will create a new license key.

```sh
curl http://localhost:8080/license_keys \
    -d public_key="unique-public-key" \
    -d public_password="unique-public-password" \
    -d license_password="unique-license-password" \
    -d name=testing \
    -d code=code \
    -X POST -i
```

Here's an example of the output from request above:

    {
        "code": "your-code",
        "id": "3bade490-defe-477d-8146-be0f621940ed",
        "license_password": "your-license-password",
        "name": "testing",
        "public_key": "your-public-key",
        "public_password": "your-public-password",
        "metadata": {},
        "valid": false
    }

Note, `public_key`, `public_password`, and `license_password` must use one-liner values.
Also it's worth noting that `metadata` is still empty and `valid` is set to `false` after adding license key.
These 2 keys will be populated after registering first consumer provider.

For details on how to register a license key, refer to [License Key API](../../reference/api/license_key/) page.

### Registering Consumer Provider
A consumer provider can be registered after the installation of the cluster and master provider.
It's worth noting that consumer provider should be hosted in another server (separated from master provider).
As usual, the `postinstall.py` script generates TLS certificates which are stored in `/etc/docker` directory.

There are few things we need to check before registering the provider.

1.  Grab the actual server hostname. We can use `hostname -f` command. In this example, we have `gluu-consumer` as the server hostname.
2.  Since Gluu Cluster relies on salt, we need to double check whether salt uses the same hostname as seen in step 1.
    We can use `cat /etc/salt/minion_id` command. If the command returns `gluu-consumer` as well, we are safe to proceed to provider registration.
    If it's not, use value from `/etc/salt/minion_id` as provider hostname.

The following command registers a provider using `curl`.

```
curl http://localhost:8080/providers \
    -d hostname=gluu-consumer \
    -d docker_base_url='https://128.199.242.75:2376' \
    -d type='consumer' \
    -X POST -i
```

The parameters of the command are explained below:

* `hostname` is the FQDN hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-consumer` package. It is recommended to use `https` scheme.
* `type` is the provider type (either `master` or `consumer`)

A successful request returns a HTTP 201 status code:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/58848b94-0671-48bc-9c94-04b0351886f1

{
    "docker_base_url": "https://128.199.242.75:2376",
    "hostname": "gluu-consumer",
    "id": "58848b94-0671-48bc-9c94-04b0351886f1",
    "type": "consumer"
}
```

The `provider_id` is required to deploy nodes, so it is best to keep the reference as an environment variable.

    export CONSUMER_PROVIDER_ID=58848b94-0671-48bc-9c94-04b0351886f1

A successful creation of provider follows a background job to complete internal IP routing through `weave` and importing necessary docker certificates.
By default, it may take up-to 25 seconds to finish the provider creation.

#### Imported docker Certificates

To deploy/delete nodes, `gluu-flask` uses docker Remote API protected by TLS, hence `gluu-flask` require necessary docker certificates.
These certificates are copied from host-level docker config directory (`/etc/docker`):

1. `/etc/docker/ca.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__ca.pem`
2. `/etc/docker/key.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__key.pem`
3. `/etc/docker/cert.pem` is imported as `/var/lib/gluu-cluster/docker_certs/$PROVIDER_ID__cert.pem`

##### Weave Routing

Run the following command to check if routing is ready:

```sh
weave ps
```
If the routing is ready then the output will be similar to the following:
```
1d34079275fc aa:d3:17:16:52:40 10.2.1.253/24
e5d1dd8d9ad4 32:7b:97:97:02:c5
```
Please use the following command to check whether `weave` in master provider ready to accept connections from other providers:

```
$ weave status
```

The command will provide a similar output as below:

```sh
$ weave status

       Version: v1.1.0

       Service: router
      Protocol: weave 1..2
          Name: d6:5f:eb:ea:b7:04(gluu-consumer)
    Encryption: enabled
 PeerDiscovery: enabled
       Targets: 0
   Connections: 1
         Peers: 2

       Service: ipam
     Consensus: achieved
         Range: [10.2.1.0-10.2.2.0)
 DefaultSubnet: 10.2.1.0/24

       Service: dns
        Domain: gluu.local.
           TTL: 1
       Entries: 5

```

Before deploying any node in consumer provider, we need to check whether license key is valid.
Any request to deploy node in any consumer provider will be rejected if license key is invalid.

We can use this command to check the license:

    curl -i http://localhost:8080/license_keys

Here's an example of output from request above:

    [
        {
            "code": "your-code",
            "id": "3bade490-defe-477d-8146-be0f621940ed",
            "license_password": "your-license-password",
            "name": "testing",
            "public_key": "your-public-key",
            "public_password": "your-public-password",
            "metadata": {
                "expiration_date": null,
                "license_count_limit": 20,
                "license_features": [
                    "gluu_server"
                ],
                "license_name": "testing-license",
                "license_type": null,
                "multi_server": true,
                "thread_count": 3
            },
            "valid": true
        }
    ]

As we can see, as `valid` key is set to `true` and `metadata` key is not set to `null` or `{}`,
we can guarantee that the license key is valid.

However there are circumtances when `valid` and `metadata` keys are not populated correctly.
The most common issue is license key having incorrect data
(either `public_key`, `license_password`,`public_password`, or `code` attribute).
To fix them, we can update the data via [License Key API](../../reference/api/license_key#update-a-license-key).


### Deploying Nodes
The deployment of nodes can be done after the creation of Cluster and Provider entities. The nodes are interdependent, it's recommended to deploy them in the following order

1. `ldap` LDAP Node

2. `oxauth` oxAuth Node

3. `nginx` nginx Node

4. optional `oxidp` oxIDP Node

#### LDAP Node

Run the following command to deploy the ldap node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$CONSUMER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

##### LDAP Replication

Since we already have another LDAP node in master provider, all LDAP nodes will replicate themselves.
However we need to check whether replication are created successfully by login to one of LDAP node.

    docker exec -ti <nodename> bash
    /opt/opendj/bin/dsreplication status

#### oxAuth Node
Run the following command to deploy oxAuth node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$CONSUMER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxauth \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

#### nginx Node
Run the following command to deploy the nginx node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$CONSUMER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=nginx \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

#### oxIdp Node (optional)
Run the following command to deploy oxIdp node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$CONSUMER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxidp \
    -X POST -i
```

A successful request returns a HTTP 202 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
The following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/node_logs/<node-name>/setup
```

## Local DNS Server

Starting from v0.4.0, Gluu Cluster uses its own local DNS server to provide service discovery between containers (nodes).
This local DNS server lists `domain_name` for all nodes in the cluster.

To see all `domain_node`, we can use a shell command:

    weave status dns

The following list is an example of output from the command above:

    97b8a9af9463.nginx      10.2.1.8    97b8a9af9463    d6:5f:eb:ea:b7:03
    baac7896cdfe.oxtrust    10.2.1.3    baac7896cdfe    d6:5f:eb:ea:b7:03
    c4812882445f.oxauth     10.2.1.1    c4812882445f    d6:5f:eb:ea:b7:03
    c7dde0b8d4e3.ldap       10.2.1.9    c7dde0b8d4e3    d6:5f:eb:ea:b7:03
    ea3e185214a4.oxidp      10.2.1.2    ea3e185214a4    d6:5f:eb:ea:b7:03

Each line is consists of columns:

1.  Short format of local `domain_name` of the node. For example, `97b8a9af9463.nginx` is a shorthand of `97b8a9af9463.nginx.gluu.local`.

    It is worth noting that this `domain_name` only resolvable inside the node. We can test connectivity between nginx node (97b8a9af9463)
    and oxtrust node (baac7896cdfe):

        $ docker exec 97b8a9af9463 ping -c 4 baac7896cdfe.oxtrust.gluu.local
        PING baac7896cdfe.oxtrust.gluu.local (10.2.1.3) 56(84) bytes of data.
        64 bytes from baac7896cdfe.oxtrust.gluu.local (10.2.1.3): icmp_seq=1 ttl=64 time=0.030 ms
        64 bytes from baac7896cdfe.oxtrust.gluu.local (10.2.1.3): icmp_seq=2 ttl=64 time=0.044 ms
        64 bytes from baac7896cdfe.oxtrust.gluu.local (10.2.1.3): icmp_seq=3 ttl=64 time=0.045 ms
        64 bytes from baac7896cdfe.oxtrust.gluu.local (10.2.1.3): icmp_seq=4 ttl=64 time=0.045 ms

        --- baac7896cdfe.oxtrust.gluu.local ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3001ms
        rtt min/avg/max/mdev = 0.030/0.041/0.045/0.006 ms

2.  `weave_ip` address of the node. We can ping the node directly from host via `weave_ip` address:

        $ ping -c 4 10.2.1.3
        PING 10.2.1.3 (10.2.1.3) 56(84) bytes of data.
        64 bytes from 10.2.1.3: icmp_seq=1 ttl=64 time=0.056 ms
        64 bytes from 10.2.1.3: icmp_seq=2 ttl=64 time=0.040 ms
        64 bytes from 10.2.1.3: icmp_seq=3 ttl=64 time=0.038 ms
        64 bytes from 10.2.1.3: icmp_seq=4 ttl=64 time=0.037 ms

        --- 10.2.1.3 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 2999ms
        rtt min/avg/max/mdev = 0.037/0.042/0.056/0.011 ms

3.  A unique `id` of the node.
4.  weave peer where the node is deployed.

## Simple Load Balancer

Given that we had master and consumer providers, we can use load balancer to manage requests
to our cluster. It's worth noting that load balancer setup is not provided by Gluu Cluster.

For simple load balancer, we can use `nginx`. Just make sure we use a separate VM to host the load balancer.

    upstream backend {
        server 128.199.242.74:443 fail_timeout=10s; # IP address of master provider
        server 128.199.242.75:443 fail_timeout=10s; # IP address of consumer provider
    }

    server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;
        server_name gluu.example.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl;
        ssl on;
        ssl_certificate /etc/nginx/certs/nginx.crt; # grab the file from gluunginx node in master provider
        ssl_certificate_key /etc/nginx/certs/nginx.key; # grab the file from gluunginx node in master provider

        server_name gluu.example.com;

        location / {
            proxy_pass https://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }
    }

Copy the example above and save it as `/etc/nginx/sites-available/gluu-balancer.conf`. Afterwards, we need to enable this new config and replace default config.

    cd /etc/nginx/sites-enable
    rm default
    ln -s ../sites-available/gluu-balancer.conf
    service nginx restart
