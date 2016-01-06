## Node API

Node is an entity represents a `docker` container.

---

### Create New Node

    POST /nodes

To create a node, provider and cluster entities must be created first.

__URL:__

    http://localhost:8080/nodes

__Form parameters:__

*   `cluster_id` (required)

    The ID of Cluster.

*   `provider_id` (required)

    The ID of Provider.

*   `node_type` (required)

    Node type (currently only supports `ldap`, `oxauth`, `oxtrust`, `nginx`, and `oxidp`).
    Note, to create a clustered nodes successfully, the nodes must be created
    in following order:

    1. `ldap`
    2. `oxauth`
    3. `oxidp` (optional; required if we want to use SAML)
    4. `nginx`
    5. `oxtrust`

    There are few rules about nodes:

    * `httpd` node is no longer supported since v0.4.0.
    * Maximum allowed `ldap` nodes are 4 per cluster.
    * There's no restriction on how many `oxauth` and `oxidp` nodes per provider or cluster.
    * Only 1 `nginx` node can be deployed in each provider.
    * Only 1 `oxtrust` node can be deployed in the cluster and it must be deployed in master provider.

*   `connect_delay` (optional)

    Time to wait (in seconds) before start connecting to node (default to 10 seconds).

*   `exec_delay` (optional)

    Time to wait (in seconds) before start executing command in node (default to 15 seconds).

__Request example:__

```sh
curl http://localhost:8080/nodes \
    -d provider_id=58848b94-0671-48bc-9c94-04b0351886f0 \
    -d cluster_id=9ea4d520-bbba-46f6-b779-c29ee99d2e9e \
    -d node_type=ldap \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 202 ACCEPTED
Content-Type: application/json
Location: http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043
X-Deploy-Log: /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043-setup.log

{
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
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
    "state": "IN-PROGRESS",
    "domain_name": ""
}
```

Since deploying a node may take awhile, the build process is running as background job.
As we can see in example above, the ``state`` is set as `IN-PROGRESS`.
Also, `id`, `ip`, and `weave_ip` are left blank.
But they will be populated eventually during the process.

There are several ways to track the deployment progress:

1.  By looking at the log file (the path is set in `X-Deploy-Log` response header above).
    A typical example is by using `tail` shell command,
    for example `tail -F /var/log/gluu/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043-setup.log`.

2.  By making requests periodically to [retrieve node resource](./#get-a-node) (the URL is set in `Location` response header above).

__Status Code:__

* `202`: Request has been accepted.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `403`: Access denied.
* `500`: The server having errors.

---

### Get A Node

    GET /nodes/{id}

__URL:__

    http://localhost:8080/nodes/{id}

__Request example:__

```sh
curl http://localhost:8080/nodes/9d99c95c4043 -i
```

or

```sh
curl http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "ldap_port": "1389",
    "ldap_admin_port": "4444",
    "ip": "172.17.0.9",
    "ldap_binddn": "cn=directory manager",
    "ldaps_port": "1636",
    "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
    "weave_ip": "10.2.1.1",
    "type": "ldap",
    "id": "9d99c95c4043",
    "ldap_jmx_port": "1689",
    "state": "SUCCESS",
    "domain_name": "9d99c95c4043.ldap.gluu.local"
}
```

There are 4 states we need to know:

1. ``IN-PROGRESS``: node is being deployed
2. ``SUCCESS``: node is successfully deployed
3. ``FAILED``: node is failed
4. ``DISABLED``: node is disabled

__Status Code:__

* `200`: Node is exist.
* `404`: Node is not exist.
* `500`: The server having errors.

---

### List All Nodes

    GET /nodes

__URL:__

    http://localhost:8080/nodes

__Request example:__

```sh
curl http://localhost:8080/nodes -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
        "name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
        "ldap_port": "1389",
        "ldap_admin_port": "4444",
        "ip": "172.17.0.9",
        "ldap_binddn": "cn=directory manager",
        "ldaps_port": "1636",
        "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
        "weave_ip": "10.2.1.1",
        "type": "ldap",
        "id": "9d99c95c4043",
        "ldap_jmx_port": "1689",
        "state": "SUCCESS",
        "domain_name": "9d99c95c4043.ldap.gluu.local"
    }
]
```

Note, if there's no node, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A Node

    DELETE /nodes/{id}

By default, node with `IN_PROGRESS` state cannot be deleted.
Any attempt to delete node with `IN_PROGRESS` state will raise status code 403.
To force node deletion, add `force_rm` option in the request (see below).

__URL:__

    http://localhost:8080/nodes/{id}

__Query string parameters:__

*   `force_rm` (optional)

    A boolean to delete the node regardless of its state. By default is set to `false`.

    * Truthy values: `1`, `True`, `true`, or `t`
    * Falsy values: `0`, `False`, `false`, or `f`

    Unknown value will be ignored.

__Request example:__

```sh
curl http://localhost:8080/nodes/9d99c95c4043 -X DELETE -i --max-time 300
```

or

```sh
curl http://localhost:8080/nodes/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 -X DELETE -i --max-time 300
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: Node has been deleted.
* `403`: Access denied.
* `404`: Node is not exist.
* `500`: The server having errors.
