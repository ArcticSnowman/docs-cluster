# Cluster Management
[TOC]

The cluster packages consisting of gluu-master, gluu-flask and gluu-consumer must be installed before configuring the cluster environment. Please see the [Installation Instructions](http://www.gluu.org/docs-cluster/admin-guide/installation/) for more details.

A [Cluster][cluster-api] is a set of nodes deployed in one or more [Providers][provider-api]. The cluster
contains information shared across providers, like hostname.

## Cluster Management
### Creating Cluster

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
    "httpd_nodes": [],
    "org_short_name": "my-org",
    "org_name": "my-org",
    "id": "1279de28-b6d0-4052-bd0c-cc46a6fd5f9f",
    "oxtrust_nodes": [],
    "name": "cluster1"
}
```

We'll need the `cluster_id` when deploying nodes, so let's keep the reference to `cluster_id` as environment variable.

```
export CLUSTER_ID=1279de28-b6d0-4052-bd0c-cc46a6fd5f9f
```

### Registering Provider
A provider must be registered after the installation of the cluster packages.
This creates the entity in the `gluu-flask` JSON database to let the  API's know where to deploy the instances.
The `postinstall.py` script generates TLS certificates which are stored in `/etc/docker` directory.

The contents of the following files should be copied for later use:

* `/etc/docker/ca.pem` (CA certificate)
* `/etc/docker/cert.pem` (TLS certificate)
* `/etc/docker/key.pem`: (TLS certificate key)

The following command registers a provider using `curl`.

```
curl http://localhost:8080/providers \
    -d hostname=gluu.example.com \
    -d docker_base_url='https://128.199.242.74:2375' \
    -d ssl_key='multi-line contents of key.pem' \
    -d ssl_cert='multi-line contents of cert.pem' \
    -d ca_cert='multi-line contents of ca.pem' \
    -d type='master' \
    -X POST -i
```

The parameters of the command are explained below:

* `hostname` is the FQDN hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-master` package. It is recommended to use `https` scheme.
* `ssl_key` is the contents of `key.pem`
* `ssl_cert` is the contents of `cert.pem`
* `ca_cert` is the contents of `ca.pem`
* `type` is the provider type (either `master` or `consumer`)

A successful request returns a HTTP 201 status code:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/58848b94-0671-48bc-9c94-04b0351886f0

{
    "docker_base_url": "https://128.199.242.74:2375",
    "hostname": "gluu.example.com",
    "id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "ssl_key": "multi-line contents of key.pem",
    "ssl_cert": "multi-line contents of cert.pem",
    "ca_cert": "multi-line contents of ca.pem",
    "type": "master"
}
```

The `provider_id` is required to deploy nodes, so it is best to keep the reference as an environment variable.
The successful creation of provider follows a background job to setup internal routing through `weave`. It may take up-to 25 seconds to finish routing.

Run the following command to check if routing is ready:

```sh
weave ps
```
If the routing is ready then the output will be similar to the following:
```
weave:expose ca:68:e4:e8:ed:26 10.2.1.254/24
e5d1dd8d9ad4 32:7b:97:97:02:c5
```
Please use the following command to check whether `weave` in master provider ready to accept connections from other providers:

```
$ weave status
```

The command will provide a similar output as below:

```sh
$ weave status
weave router 0.10.0
Encryption on
Our name is 42:8d:28:15:14:eb(gluu.example.com)
Sniffing traffic on &{300 65535 ethwe ee:89:e5:75:54:2c up|broadcast|multicast}
MACs:
Peers:
42:8d:28:15:14:eb(gluu.example.com) (v6) (UID 5470999222361683535)
Routes:
unicast:
42:8d:28:15:14:eb -> 00:00:00:00:00:00
broadcast:
42:8d:28:15:14:eb -> []
Reconnects:

```

### Deploying Nodes
The deployment of nodes can be done after the creation of Cluster and Provider entities. The nodes are interdependent, it's recommended to deploy them in the following order

1. `ldap` LDAP Node

2. `oxauth` oxAuth Node

3. `oxtrust` oxTrust Node

4. `httpd` Apache Node

#### LDAP Node

Run the following command to deploy the ldap node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request returns a HTTP 201 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
Run the following command to tail the log file:

```
tail -F /var/log/gluu/<node-name>-setup.log
```

Alternatively, the following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/nodes/<node-name>
```

#### oxAuth Node
Run the following command to deploy oxAuth node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxauth \
    -X POST -i
```

A successful request returns a HTTP 201 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
Run the following command to tail the log file:

```
tail -F /var/log/gluu/<node-name>-setup.log
```

Alternatively, the following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/nodes/<node-name>
```

#### oxTrust Node
Run the following command to deploy oxTrust node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=oxtrust \
    -X POST -i
```
A successful request returns a HTTP 201 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
Run the following command to tail the log file:

```
tail -F /var/log/gluu/<node-name>-setup.log
```

Alternatively, the following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/nodes/<node-name>
```

#### Apache Node
Run the following command to deploy the httpd node:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=httpd \
    -X POST -i
```

A successful request returns a HTTP 201 status code. Note the node name from the code.
The progress of deployment can be followed by tailing the log.
Run the following command to tail the log file:

```
tail -F /var/log/gluu/<node-name>-setup.log
```

Alternatively, the following command can be used periodically to check the deployment of the node:

```
curl http://localhost:8080/nodes/<node-name>
```

### Access oxTrust Web Interface

The Gluu Server web interface can be accessed after the deployment of the nodes are complete. The oxTrust UI is run at `https://localhost:8443` in the master provider. To access the UI, `ssh` tunneling must be used. The exaple will use `128.199.242.74` as an example IP address of the master provider.

```
ssh -L 8443:localhost:8443 root@128.199.242.74
```

After the connection to 128.199.242.74:8443 is established, visit `https://localhost:8443` in the web browser.
The oxTrust login page will load asking for a username and a password.

* username: `admin`
* password: `admin_pw` value from the LDAP node

If the credentials are supplied correctly, the page will be redirected to the oxTrust dashboard.