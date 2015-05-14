## Provider API

Provider is an entity represents a machine where `docker`, `salt-minion`,
and `weave` are hosted. This machine could be a Gluu Cluster Master or
Gluu Cluster Consumer.

---

### Create New Provider

`POST /provider`

NOTE: make sure `docker`, `salt-minion`, and `weave` have been [installed and configured](../../admin-guide/installation/package.md).


__URL:__

`http://localhost:8080/provider`

__Form parameters:__

*   `hostname` (required)

    Host name of the machine. When in doubt, use `hostname --long` shell command to get the name.

*   `docker_base_url` (required)

    Docker remote API URL. Supported format is `$IP:$PORT`, e.g. `128.199.198.172:2375`.
    This assumes `docker` daemon has been configured to listen to TCP connection.
    See [configuring docker daemon](../../admin-guide/installation/package.md#docker) for example.

__Request example:__

```sh
curl http://localhost:8080/provider \
    -d hostname=my-host \
    -d docker_base_url='128.199.198.172:2375' \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/provider/283bfa41-2121-4433-9741-875004518677

{
    "docker_base_url": "128.199.198.172:2375",
    "hostname": "my-host",
    "id": "249c282f-0490-48f3-8575-c671dfb7b618"
}
```

__Status Code:__

* `201`: Provider is successfully created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Get A Provider

`GET /provider/{id}`

__URL:__

`http://localhost:8080/provider/{id}`

__Request example:__

```sh
curl http://localhost:8080/provider/283bfa41-2121-4433-9741-875004518677 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "docker_base_url": "128.199.198.172:2375",
    "hostname": "my-host",
    "id": "283bfa41-2121-4433-9741-875004518677"
}
```

__Status Code:__

* `200`: Provider is exist.
* `404`: Provider is not exist.
* `500`: The server having errors.

---

### List All Providers

`GET /provider`

__URL:__

`http://localhost:8080/provider`

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
        "docker_base_url": "128.199.198.172:2375",
        "hostname": "my-host",
        "id": "283bfa41-2121-4433-9741-875004518677"
    }
]
```

Note, if there's no provider, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A Provider

`DELETE /provider/{id}`

__URL:__

`http://localhost:8080/provider/{id}`

__Request example:__

```sh
curl http://localhost:8080/provider/283bfa41-2121-4433-9741-875004518677 \
    -X DELETE -i
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
