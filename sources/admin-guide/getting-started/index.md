## Getting Started

## Pre-requisites

1. Gluu Server Master packages are already installed. Refer to [Package](../installation/package.md) documentation for details.
2. In this example, we'll use `gluu.example.com` as hostname and `128.199.242.74` as IP address from `eth0` interface.

## Registering Provider

In Gluu jargon, a "Provider" is server where `docker`, `weave`, and `salt-minion` are hosted.
Both `gluu-master` and `gluu-consumer` are Providers.

When you first deploy a Gluu Cluster, you will start with the `gluu-master` package. After you install the
packages, you need to "register" Provider. This creates the entity in the gluu-flask json database. You
need to do this so that the API's know where to deploy your instances.

Note: `gluu-flask` api's should only be run on localhost, and accessed locally, as there is no
api protection in place at this time.

```
curl http://localhost:8080/provider \
    -d hostname=gluu.example.com \
    -d docker_base_url='128.199.242.74:2375' \
    -X POST -i
```

Here's a brief explanation of parameters used in the command above:

* `hostname` is the hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-master` package.

A successful request will returns a response (with HTTP status code 201) like this:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/provider/249c282f-0490-48f3-8575-c671dfb7b618

{
    "docker_base_url": "128.199.242.74:2375",
    "hostname": "gluu.example.com",
    "id": "249c282f-0490-48f3-8575-c671dfb7b618"
}
```

We'll need the `provider_id` when deploying nodes, so let's keep the reference to `provider_id` as environment variable.

```
export PROVIDER_ID=249c282f-0490-48f3-8575-c671dfb7b618
```

## Creating Cluster

A [Cluster][cluster-api] is a set of nodes deployed in one or more [Providers][provider-api]. The cluster
contains information shared across providers, like hostname. Only create one cluster.
Here is a sample using `curl`:

```
curl http://localhost:8080/cluster \
    -d name=cluster1 \
    -d org_name=my-org \
    -d org_short_name=my-org \
    -d city=Austin \
    -d state=TX \
    -d country_code=US \
    -d admin_email='info@example.com' \
    -d ox_cluster_hostname=ox.example.com \
    -d weave_ip_network=10.2.1.0/24 \
    -d admin_pw=secret \
    -X POST -i
```

Here's a brief explanation of parameters used in command above:

* `name` is cluster name and currently it doesn't mean anything other than labeling.
* `org_name`, `org_short_name`, `city`, `state`, `country_code`, and `admin_email` will be used for X509 certificate.
* `ox_cluster_hostname` is a URL for web UI; make sure the name is reachable from browser.
* `weave_ip_network` is IP address network used for inter-container communication; Make we use unique IP address per cluster.
* `admin_pw` is used for LDAP password, LDAP replication password, and oxTrust admin password.

A successful request returns a HTTP 201 status code:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/cluster/1279de28-b6d0-4052-bd0c-cc46a6fd5f9f

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
    "ox_cluster_hostname": "ox.example.com",
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

[cluster-api]: ../../reference/api/cluster.md
[provider-api]: ../../reference/api/provider.md

## Deploying Nodes

Once we had Cluster and Provider entities, we can start deploying nodes. Note, as each nodes is depends on each other, it's recommended to deploy nodes in following order:

1. `ldap` (LDAP node)
2. `oxauth` (oxAuth node)
3. `oxtrust` (oxTrust node)
4. `httpd` (Apache node)

Let's start deploying `ldap` node first. Type the command below in the shell:

```
curl http://localhost:8080/node \
    -d provider_id=$PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request returns a response (with HTTP status code 202) like this:

```
HTTP/1.0 202 ACCEPTED
Content-Type: application/json

{
    "log": "/tmp/gluuopendj-build-OoQ7TM.log"
}
```

Since deploying a node may take awhile, it's recommended to follow the progress via its log file.
Notice the `{"log": "/tmp/gluuopendj-build-OoQ7TM.log"}` in JSON response above?
Now we can use shell command to follow the progress.

Type the command below:

```
tail -F /tmp/gluuopendj-build-OoQ7TM.log
```

The log file will inform whether the node deployment is succeed or failed.

If `ldap` node is successfully deployed, we can continue deploying nodes for `oxauth`, `oxtrust`, `httpd` sequentially. Repeat the `curl` and `tail` command above, but make sure we're using different value for `node_type` parameter.

## Accessing oxTrust Web UI using Browser

After all nodes successfully deployed, we can accessing the web UI using browser. Remember the `ox_cluster_hostname` parameter? Since we set `ox.example.com` as its value, open the web browser and type `https://ox.example.com` in address bar. The application will take us to login page. Enter the following values in the form fields:

* `admin` as username
* `secret` as password (taken from `admin_pw` value when we created the cluster earlier)

A successful login will be redirected to oxTrust dashboard.

## Adding Additional Provider

As we have mentioned before, both `gluu-master` and `gluu-consumer` are Providers.
In the example above, we already have a master provider.
We may want to add extra consumer providers, for example to have LDAP replication.

In this example, there are pre-requisites:

1. Gluu Server Consumer package is already installed. Refer to [Package](../installation/package.md) documentation for details.
2. We'll use `gluu2.example.com` as hostname and `128.199.242.75` as IP address from `eth0` interface.

Let's add this new provider:

```
curl http://localhost:8080/provider \
    -d hostname=gluu2.example.com \
    -d docker_base_url='128.199.242.75:2375' \
    -X POST -i
```

A successful request will returns a response (with HTTP status code 201) like this:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/provider/249c282f-0490-48f3-8575-c671dfb7b619

{
    "docker_base_url": "128.199.242.75:2375",
    "hostname": "gluu2.example.com",
    "id": "249c282f-0490-48f3-8575-c671dfb7b619"
}
```

We'll need the `provider_id` when deploying nodes, so let's keep the reference to `provider_id` as environment variable.

```
export PROVIDER2_ID=249c282f-0490-48f3-8575-c671dfb7b619
```

Afterwards, deploy a new LDAP node. This new LDAP node will be replicated since we already have an existing LDAP node.

```
curl http://localhost:8080/node \
    -d provider_id=$PROVIDER2_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request returns a response (with HTTP status code 202) like this:

```
HTTP/1.0 202 ACCEPTED
Content-Type: application/json

{
    "log": "/tmp/gluuopendj-build-OoQ7TN.log"
}
```

Follow the progress of node deployment:

```
tail -F /tmp/gluuopendj-build-OoQ7TN.log
```

The log file will inform whether the new LDAP node deployment (including replication process) is succeed or failed.

## Using the Swagger UI

gluu-flask includes Swagger for all API's, and the SwaggerUI to browse and interact with the APIs. 
The url is [http://127.0.0.1:8080/api/spec.html](http://127.0.0.1:8080/api/spec.html)