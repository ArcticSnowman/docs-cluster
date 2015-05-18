## Getting Started

This article assumes few things:

1. The server is using Ubuntu 14.04 (as it's the only supported OS right now)
2. We're using single server/host setup which only requires Gluu Cluster Master and Gluu Cluster API packages.
3. We already installed and configured Gluu Cluster Packages. Refer to [Package](../installation/package.md) documentation for details.
4. We set `my-gluu-master` as hostname and `128.199.198.172` as IP address from `eth0` interface.

### Creating Cluster

[Cluster][cluster-api] is a set of nodes deployed in one or more [Providers][provider-api]. Since Glu Cluster API only supports a single cluster for now, we're going to create a new one. Gluu Cluster API itself is a HTTP REST API, so using any HTTP client (including `curl`) will work.

Type command below in the shell:

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

A successful request will returns a response (with HTTP status code 201) like this:

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

We'll need `cluster_id` when deploying nodes, so let's keep the reference to `cluster_id` as environment variable.

```
export CLUSTER_ID=1279de28-b6d0-4052-bd0c-cc46a6fd5f9f
```

[cluster-api]: ../reference/api/cluster.md
[provider-api]: ../reference/api/provider.md

### Registering Provider

Provider is an entity represents a server/host where `docker`, `weave`, and `salt-minion` are hosted.
For example, if we install `gluu-master` or `gluu-consumer` package in a server/host, then it is considered as a provider.

Since this article assumes we only use `gluu-master` package, let's register the host as provider. Again, let's type command below in the shell:

```
curl http://localhost:8080/provider \
    -d hostname=my-gluu-master \
    -d docker_base_url='128.199.198.172:2375' \
    -X POST -i
```

Here's a brief explanation of parameters used in command above:

* `hostname` is the hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-master` package.

A successful request will returns a response (with HTTP status code 201) like this:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/provider/249c282f-0490-48f3-8575-c671dfb7b618

{
    "docker_base_url": "128.199.198.172:2375",
    "hostname": "my-gluu-master",
    "id": "249c282f-0490-48f3-8575-c671dfb7b618"
}
```

We'll need `provider_id` when deploying nodes, so let's keep the reference to `provider_id` as environment variable.

```
export PROVIDER_ID=249c282f-0490-48f3-8575-c671dfb7b618
```

### Deploying Nodes

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

### Accessing Web UI using Browser

After all nodes successfully deployed, we can accessing the web UI using browser. Remember the `ox_cluster_hostname` parameter? Since we set `ox.example.com` as its value, open the web browser and type `https://ox.example.com` in address bar. The application will take us to login page. Enter the following values in the form fields:

* `admin` as username
* `secret` as password (taken from `admin_pw` value when we created the cluster earlier)

A successful login will be redirected to oxTrust dashboard.
