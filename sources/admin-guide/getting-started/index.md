## Getting Started

## Pre-requisites

1. Gluu Server Master packages are already installed. Refer to [Package](../installation/package.md) documentation for details.
2. In this example, we'll use `gluu.example.com` as hostname and `128.199.242.74` as IP address from `eth0` interface.

Note: `gluu-flask` API's should only be run on localhost, and accessed locally, as there is no
api protection in place at this time.

## Creating Cluster

A [Cluster][cluster-api] is a set of nodes deployed in one or more [Providers][provider-api]. The cluster
contains information shared across providers, like hostname. Only create one cluster.
Here is a sample using `curl`:

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

Here's a brief explanation of parameters used in command above:

* `name` is cluster name and currently it doesn't mean anything other than labeling.
* `org_name`, `org_short_name`, `city`, `state`, `country_code`, and `admin_email` will be used for X509 certificate.
* `ox_cluster_hostname` is a URL for the appliance; make sure the name is reachable from browser.
* `weave_ip_network` is IP address network used for inter-container communication; Make we use unique IP address per cluster.
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

## Registering Provider

In Gluu jargon, a "Provider" is server where `docker`, `weave`, and `salt-minion` are hosted.
Both `gluu-master` and `gluu-consumer` are Providers.

When you first deploy a Gluu Cluster, you will start with the `gluu-master` package. After you install the
packages, you need to "register" Provider. This creates the entity in the `gluu-flask` JSON database. You
need to do this so that the API's know where to deploy your instances. It's worth noting that a cluster
must be created first before start registering any provider.

As of `v0.2`, it's possible to use a secure `docker` daemon. When we use `postinstall.py` script, TLS certificates
are generated and saved in `/etc/docker` directory.

Copy the contents of following files for later use:

* `/etc/docker/ca.pem` (CA certificate)
* `/etc/docker/cert.pem` (TLS certificate)
* `/etc/docker/key.pem`: (TLS certificate key)

Now we can register a provider:

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

Here's a brief explanation of parameters used in the command above:

* `hostname` is the FQDN hostname of the server/host.
* `docker_base_url` is the Docker API URL configured after installing `gluu-master` package.
* `ssl_key` is the contents of `key.pem`
* `ssl_cert` is the contents of `cert.pem`
* `ca_cert` is the contents of `ca.pem`
* `type` is the provider type (either `master` or `consumer`)

A successful request will returns a response (with HTTP status code 201) like this:

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

We'll need the `provider_id` when deploying nodes, so let's keep the reference to `provider_id` as environment variable.

```
export MASTER_PROVIDER_ID=58848b94-0671-48bc-9c94-04b0351886f0
```

After provider successfully created, a background job takes place to setup internal routing through `weave`.
It may takes approximately 25 seconds to finish the setup.

To check whether internal routing is ready, we can use `weave ps` command in the shell:

```sh
weave ps
```

If routing is ready, the output will look like the snippet below:

```
weave:expose ca:68:e4:e8:ed:26 10.2.1.254/24
e5d1dd8d9ad4 32:7b:97:97:02:c5
```

To check whether `weave` in master provider ready to accept connection from other providers,
we can use `weave status` command in the shell:

```
$ weave status
```

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

Notice that there's `gluu.example.com` peer? Since we don't have other providers,
Peers only lists our own master provider. We will add additional (consumer) providers
in the next section, but before that, let's start deploying nodes in master provider.

## Deploying Nodes

Once we had Cluster and Provider entities, we can start deploying nodes. Note, as each nodes is depends on each other, it's recommended to deploy nodes in following order:

1. `ldap` (LDAP node)
2. `oxauth` (oxAuth node)
3. `oxtrust` (oxTrust node)
4. `httpd` (Apache node)

Let's start deploying `ldap` node first. Type the command below in the shell:

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$MASTER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request will returns a response (with HTTP status code 202):

```http
HTTP/1.0 202 ACCEPTED
Content-Type: application/json
Location: http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042
X-Deploy-Log: /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042-setup.log

{
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042",
    "ldap_port": "1389",
    "ldap_admin_port": "4444",
    "ip": "",
    "ldap_binddn": "cn=directory manager",
    "ldaps_port": "1636",
    "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
    "weave_ip": "",
    "type": "ldap",
    "id": "",
    "ldap_jmx_port": "1689",
    "state": "IN-PROGRESS"
}
```

Since deploying a node may take awhile, it's recommended to follow the progress via its log file.
Notice the `X-Deploy-Log: /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042-setup.log`,
in response header above?
Now we can use shell command to follow the progress.

Type the command below:

```
tail -F /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042-setup.log
```

The log file will inform whether the node deployment is succeed or failed.

Another alternative is to make requests periodically to retrieve node resource. Since node can be retrieved by its `id` or `name`, we can use `curl` command:

```
curl http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59042
```

If `ldap` node is successfully deployed, we can continue deploying nodes for `oxauth`, `oxtrust`, `httpd` sequentially. Repeat the `curl` and `tail` command above, but make sure we're using different value for `node_type` parameter.

## Accessing oxTrust Web UI using Browser

After all nodes successfully deployed, we can accessing the oxTrust web UI using browser.
Starting from v0.3.1-14, for security reason, oxTrust is never exposed for public.
It's always run at `https://localhost:8443`.

To access the oxTrust UI, we need to do SSH tunneling. From example above, we already know
that oxTrust has been deployed to master provider (IP address is 128.199.242.74),
hence we can type the following command in our shell:

```
ssh -L 8443:localhost:8443 root@128.199.242.74
```

After the connection to 128.199.242.74:8443 is established, visit https://localhost:8443 in web browser.
The application will take us to login page. Enter the following values in the form fields:

* `admin` as username
* `secret` as password (taken from `admin_pw` value when we created the cluster earlier)

A successful login will be redirected to oxTrust dashboard.

Remember, as we can deploy oxTrust node to any provider (not just the master provider),
we can pick any oxTrust node in the cluster and do the SSH tunneling.

## Adding Additional Provider

We may want to add one or more consumer providers to achieve highly-available setup.

### Adding License Key

Before start adding one or more consumer providers, a license key must exist. Please contact support@gluu.org about license key.

```sh
curl http://localhost:8080/license_keys \
    -d public_key='my-public-key' \
    -d public_password='my-public-password' \
    -d license_password='my-license-password' \
    -d name=my-gluu-license \
    -d code=my-license-code \
    -X POST -i
```

A successful request will returns a response (with HTTP status code 201) like this:

```http
HTTP/1.0 201 CREATED
Location: http://localhost:8080/license_keys/3bade490-defe-477d-8146-be0f621940ed
Content-Type: application/json

{
    "code": "my-license-code",
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "my-license-password",
    "name": "my-gluu-license",
    "public_key": "my-public-key",
    "public_password": "my-public-password",
    "valid": false,
    "metadata": {}
}
```

NOTE: there's a known limitation with initial `valid` and `metadata` values. See [Create New License Key](../../reference/api/license_key#create-new-license-key) section in License Key API.

### Registering Consumer Provider

After we have master provider and license key, we can add one or more consumer providers. The maximum number of how many consumer providers we can register is based on license key we have created earlier.

```
curl http://localhost:8080/providers \
    -d hostname=gluu.consumer.example.com \
    -d docker_base_url='https://128.199.242.75:2375' \
    -d ssl_key='multi-line contents of consumer key.pem' \
    -d ssl_cert='multi-line contents of consumer cert.pem' \
    -d ca_cert='multi-line contents of consumer ca.pem' \
    -d type='consumer' \
    -X POST -i
```

A successful request will returns a response (with HTTP status code 201) like this:

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/58848b94-0671-48bc-9c94-04b0351886f1

{
    "docker_base_url": "https://128.199.242.75:2375",
    "hostname": "gluu.consumer.example.com",
    "id": "58848b94-0671-48bc-9c94-04b0351886f1",
    "ssl_key": "multi-line contents of key.pem",
    "ssl_cert": "multi-line contents of cert.pem",
    "ca_cert": "multi-line contents of ca.pem",
    "type": "consumer",
}
```

We'll need the `provider_id` when deploying nodes, so let's keep the reference to `provider_id` as environment variable.

```
export CONSUMER_PROVIDER_ID=58848b94-0671-48bc-9c94-04b0351886f1
```

After provider successfully created, a background job takes place to setup internal routing through `weave`.
It may takes approximately 25 seconds to finish the setup.

To check whether internal routing is ready, we can use `weave ps` command in the shell:

```sh
weave ps
```

If routing is ready, the output will look like the snippet below:

```
weave:expose ca:68:e4:e8:ed:45 10.2.1.254/24
e5d1dd8d9ad4 32:7b:97:97:02:c5
```

To check whether `weave` in consumer provider connected to master provider,
we can use `weave status` command in the shell:

```
$ weave status
weave router 0.10.0
Encryption on
Our name is ca:68:e4:e8:ed:45(gluu.consumer.example.com)
Sniffing traffic on &{46 65535 ethwe 32:7b:97:97:02:c5 up|broadcast|multicast}
MACs:
Peers:
42:8d:28:15:14:eb(gluu.example.com) (v5) (UID 5470999222361683535)
    -> ca:68:e4:e8:ed:26(gluu.consumer.example.com) [128.199.242.75:46124]
ca:68:e4:e8:ed:45(gluu.consumer.example.com) (v2) (UID 3882689006152107332)
    -> 42:8d:28:15:14:eb(gluu.example.com) [128.199.242.74:6783]
Routes:
unicast:
ca:68:e4:e8:ed:26 -> 00:00:00:00:00:00
42:8d:28:15:14:eb -> 42:8d:28:15:14:eb
broadcast:
ca:68:e4:e8:ed:26 -> [42:8d:28:15:14:eb]
42:8d:28:15:14:eb -> []
Reconnects:

```

As we can see, Peers lists `gluu.example.com` (our master provider) and `gluu.consumer.example.com` (consumer).
That means the connection between master and consumer provider has been established.
Now we can start deploying nodes in consumer provider.

### Deploying Nodes to Consumer Provider

There's no major difference between deploying nodes to master nor consumer provider.
We only need to pass the `provider_id` value pointed either master or consumer.

```sh
curl http://localhost:8080/nodes \
    -d provider_id=$CONSUMER_PROVIDER_ID \
    -d cluster_id=$CLUSTER_ID \
    -d node_type=ldap \
    -X POST -i
```

A successful request will returns a response (with HTTP status code 202):

```http
HTTP/1.0 202 ACCEPTED
Content-Type: application/json
Location: http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043
X-Deploy-Log: /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043-setup.log

{
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f1",
    "name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "ldap_port": "1389",
    "ldap_admin_port": "4444",
    "ip": "",
    "ldap_binddn": "cn=directory manager",
    "ldaps_port": "1636",
    "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
    "weave_ip": "",
    "type": "ldap",
    "id": "",
    "ldap_jmx_port": "1689",
    "state": "IN-PROGRESS"
}
```

Since deploying a node may take awhile, it's recommended to follow the progress via its log file.
Notice the `X-Deploy-Log: /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043-setup.log`
in response header above?
Now we can use shell command to follow the progress.

Type the command below:

```
tail -F /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043-setup.log
```

The log file will inform whether the node deployment is succeed or failed.

Another alternative is to make requests periodically to retrieve node resource. Since node can be retrieved by its `id` or `name`, we can use `curl` command:

```
curl http://localhost:8080/node/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043
```

If `ldap` node is successfully deployed, we can continue deploying nodes for `oxauth`, `oxtrust`, `httpd` sequentially. Repeat the `curl` and `tail` command above, but make sure we're using different value for `node_type` parameter.
