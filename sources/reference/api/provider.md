## Provider API

Provider is an entity represents a machine where `docker`, `salt-minion`,
and `weave` are hosted. This machine could be a Gluu Cluster Master or
Gluu Cluster Consumer.

---

### Create New Provider

    POST /providers

NOTE: make sure `docker`, `salt-minion`, and `weave` have been [installed and configured](../../admin-guide/installation/index.md). Also, a cluster must be created beforehand.


__URL:__

    http://localhost:8080/providers

__Form parameters:__

*   `hostname` (required)

    Host name of the machine. When in doubt, use `hostname --long` shell command to get the name.

*   `docker_base_url` (required)

    Docker remote API URL. Supported format is `unix` socket (e.g. `unix:///var/run/docker.sock`) or
    `https://$IP:$PORT`, e.g. `https://128.199.198.172:2376`.
    The latter format assumes `docker` daemon has been configured to listen to TCP connection.
    See [configuring docker daemon](../../admin-guide/installation/index.md) for example.

*   `type` (required)

    Provider type, must be one of `master` or `consumer`.

*   `connect_delay` (optional)

    Time to wait (in seconds) before start connecting to provider (default to 10 seconds).

*   `exec_delay` (optional)

    Time to wait (in seconds) before start executing command in provider (default to 15 seconds).

#### Master Provider

NOTE: Currently only supports a single `master` provider.

__Request example:__

```sh
curl http://localhost:8080/providers \
    -d hostname=master-host \
    -d docker_base_url='https://128.199.198.172:2376' \
    -d type='master' \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518677

{
    "docker_base_url": "128.199.198.172:2376",
    "hostname": "master-host",
    "id": "283bfa41-2121-4433-9741-875004518677",
    "type": "master"
}
```

#### Consumer Provider

There are prerequisites before creating a `consumer`:

1. `master` provider must exist.
2. Must have a license key. See [License Key API](./license_key.md) for details.

It's worth noting that when license for `consumer` provider is expired,
server will try to retrieve new license automatically. If succeed, the provider will use new license.
Otherwise, all `oxauth` nodes deployed on this provider will be disabled from cluster.

__Request example:__

```sh
curl http://localhost:8080/providers \
    -d hostname=consumer-host \
    -d docker_base_url='128.199.198.173:2376' \
    -d type=consumer \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518678

{
    "docker_base_url": "128.199.198.173:2376",
    "hostname": "consumer-host",
    "id": "283bfa41-2121-4433-9741-875004518678",
    "type": "consumer"
}
```

__Status Code:__

* `201`: Provider is successfully created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `403`: Access denied. Refer to message key in JSON response for details.
* `422`: Unprocessable entity. Possibly unable to retrieve license from license server.
* `500`: The server having errors.

---

### Get A Provider

    GET /providers/{id}

__URL:__

    http://localhost:8080/providers/{id}

__Request example:__

```sh
curl http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518677 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "docker_base_url": "128.199.198.172:2376",
    "hostname": "master-host",
    "id": "283bfa41-2121-4433-9741-875004518677",
    "type": "master"
}
```

__Status Code:__

* `200`: Provider is exist.
* `404`: Provider is not exist.
* `500`: The server having errors.

---

### List All Providers

    GET /providers

__URL:__

    http://localhost:8080/providers

__Request example:__

```sh
curl http://localhost:8080/provider -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "docker_base_url": "128.199.198.172:2376",
        "hostname": "master-host",
        "id": "283bfa41-2121-4433-9741-875004518677",
        "type": "master"
    },
    {
        "docker_base_url": "128.199.198.173:2376",
        "hostname": "consumer-host",
        "id": "283bfa41-2121-4433-9741-875004518678",
        "type": "consumer"
    }
]
```

Note, if there's no provider, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A Provider

    DELETE /providers/{id}

__URL:__

    http://localhost:8080/providers/{id}

__Request example:__

```sh
curl http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518677 -X DELETE -i
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: Provider has been deleted.
* `403`: Access denied. Refer to `message` key in JSON response for details.
* `404`: Provider is not exist.
* `500`: The server having errors.

---

### Update A Provider

    PUT /providers/{id}

__URL:__

    http://localhost:8080/providers/{id}

__Form parameters:__

*   `hostname` (required)

    Host name of the machine. When in doubt, use `hostname --long` shell command to get the name.

*   `docker_base_url` (required)

    Docker remote API URL. Supported format is `unix` socket (e.g. `unix:///var/run/docker.sock`) or
    `https://$IP:$PORT`, e.g. `https://128.199.198.172:2376`.
    The latter format assumes `docker` daemon has been configured to listen to TCP connection.
    See [configuring docker daemon](../../admin-guide/installation/index.md#docker) for example.

*   `ssl_key` (required only when `docker_base_url` uses HTTPS)

    The contents of `/etc/docker/key.pem` located in provider host.
    Starting from v0.4.1, this parameter is ignored.

*   `ssl_cert` (required only when `docker_base_url` uses HTTPS)

    The contents of `/etc/docker/cert.pem` located in provider host.
    Starting from v0.4.1, this parameter is ignored.

*   `ca_cert` (required only when `docker_base_url` uses HTTPS)

    The contents of `/etc/docker/ca.pem` located in provider host.
    Starting from v0.4.1, this parameter is ignored.

#### Master Provider

__Request example:__

```sh
curl http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518677 \
    -d hostname=master-host \
    -d docker_base_url='https://128.199.198.172:2376' \
    -X PUT -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "docker_base_url": "128.199.198.172:2376",
    "hostname": "master-host",
    "id": "283bfa41-2121-4433-9741-875004518677",
    "type": "master"
}
```

#### Consumer Provider

__Request example:__

```sh
curl http://localhost:8080/providers/283bfa41-2121-4433-9741-875004518678 \
    -d hostname=consumer-host \
    -d docker_base_url='128.199.198.173:2376' \
    -X PUT -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "docker_base_url": "128.199.198.173:2376",
    "hostname": "consumer-host",
    "id": "283bfa41-2121-4433-9741-875004518678",
    "type": "consumer"
}
```

__Status Code:__

* `200`: Request succeed.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `404`: Provider not found.
* `500`: The server having errors.
