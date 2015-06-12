## Node API

Node is an entity represents a `docker` container.

---

### Create New Node

`POST /node`

Note, to create a node, provider and cluster entities must be created first.

__URL:__

`http://localhost:8080/node`

__Form parameters:__

*   `cluster_id` (required)

    The ID of Cluster.

*   `provider_id` (required)

    The ID of Provider.

*   `node_type` (required)

    Node type (currently only supports `ldap`, `oxauth`, `oxtrust`, and `httpd`).
    Note, to create a clustered nodes successfully, the nodes must be created
    in following order:

    1. `ldap`
    2. `oxauth`
    3. `oxtrust`
    4. `httpd`

*   `connect_delay` (optional)

    Time to wait (in seconds) before start connecting to node (default to 10 seconds).

*   `exec_delay` (optional)

    Time to wait (in seconds) before start executing command in node (default to 15 seconds).

__Request example:__

```sh
curl http://localhost:8080/node \
    -X POST -i \
    -d provider_id=58848b94-0671-48bc-9c94-04b0351886f0 \
    -d cluster_id=9ea4d520-bbba-46f6-b779-c29ee99d2e9e \
    -d node_type=ldap
```

__Response example:__

```http
HTTP/1.0 202 ACCEPTED
Content-Type: application/json

{
    "log": "/tmp/gluuopendj-build-OoQ7TM.log"
}
```

Since building a node may take awhile, the build process is running as background job.
To follow the build process, it is recommended to see the log file as shown
in response example above. A typical example is by using `tail` shell command,
for example `tail -F /tmp/gluuopendj-build-OoQ7TM.log`.

__Status Code:__

* `202`: Request has been accepted.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Get A Node

`GET /node/{id}`

__URL:__

`http://localhost:8080/node/{id}`

__Request example:__

```sh
curl http://localhost:8080/node/9d99c95c4043 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
    "name": "gluuopendj_9ea4d520-bbba-46f6-b779-c29ee99d2e9e_988",
    "ldap_port": "1389",
    "ldap_admin_port":
    "4444", "ip": "172.17.0.9",
    "ldap_binddn": "cn=directory manager",
    "ldaps_port": "1636",
    "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
    "weave_ip": "10.2.1.1",
    "type": "ldap",
    "id": "9d99c95c4043",
    "ldap_jmx_port": "1689"
}
```

__Status Code:__

* `200`: Node is exist.
* `404`: Node is not exist.
* `500`: The server having errors.

---

### List All Nodes

`GET /node`

__URL:__

`http://localhost:8080/node`

__Request example:__

```sh
curl http://localhost:8080/node -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "provider_id": "58848b94-0671-48bc-9c94-04b0351886f0",
        "name": "gluuopendj_9ea4d520-bbba-46f6-b779-c29ee99d2e9e_988",
        "ldap_port": "1389",
        "ldap_admin_port":
        "4444", "ip": "172.17.0.9",
        "ldap_binddn": "cn=directory manager",
        "ldaps_port": "1636",
        "cluster_id": "9ea4d520-bbba-46f6-b779-c29ee99d2e9e",
        "weave_ip": "10.2.1.1",
        "type": "ldap",
        "id": "9d99c95c4043",
        "ldap_jmx_port": "1689"
    }
]
```

Note, if there's no node, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A Node

`DELETE /node/{id}`

__URL:__

`http://localhost:8080/node/{id}`

__Request example:__

```sh
curl http://localhost:8080/node/9d99c95c4043 -X DELETE -i --max-time 300
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: Node has been deleted.
* `404`: Node is not exist.
* `500`: The server having errors.
