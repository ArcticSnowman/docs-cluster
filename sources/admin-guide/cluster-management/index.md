# Cluster Management
[TOC]

The cluster packages consisting of gluu-master, gluu-flask and gluu-consumer must be installed before configuring the cluster environment. Please see the [Installation Instructions](http://www.gluu.org/docs-cluster/admin-guide/installation/) for more details.

A [Cluster][cluster-api] is a set of nodes deployed in one or more [Providers][provider-api]. The cluster
contains information shared across providers, like hostname.

## Cluster Management
### Registering Provider
A provider must be registered after the installation of the cluster packages.

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
The parameters of the command is explained below:

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

[cluster-api]: ../../reference/api/cluster.md
[provider-api]: ../../reference/api/provider.md
```

## Cluster Management with Web Interface
Gluu Cluster comes with a `gluu-webui` package to manage the cluster resources using a web interface. Please see the [Web UI](http://www.gluu.org/docs-cluster/admin-guide/webui/) page for more details.
