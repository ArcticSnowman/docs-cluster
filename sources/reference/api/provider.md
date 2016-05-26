[TOC]

## Provider API

### Overview

Provider is an entity represents a service that provides server. An example of provider is a cloud service.
Currently, there are various supported provider types:

1. [DigitalOcean](https://www.digitalocean.com/).
2. Generic; this is a _Bring Your Own Provider_ style. This should be used if none of any provider mentioned above is selected.

---

### Create New Provider

Any supported provider type can be created by sending a request to `/providers/{type}` URL, where `{type}` is the name of provider type mentioned above.

#### DigitalOcean Provider

    POST /providers/digitalocean

__URL:__

    http://localhost:8080/providers/digitalocean

__Form parameters:__

*   `name` (required)

    A unique name of the provider.

*   `digitalocean_access_token` (required)

    DigitalOcean access token.

*   `digitalocean_backups`

    Enable or disable backup for DigitalOcean droplet (turned off by default).

*   `digitalocean_private_networking`

    Enable or disable private networking for DigitalOcean droplet (turned off by default).

*   `digitalocean_region` (required)

    Region where droplet is hosted.

    Supported region:

    * `nyc1`: New York 1
    * `nyc2`: New York 2
    * `nyc3`: New York 3
    * `ams2`: Amsterdam 2
    * `ams3`: Amsterdam 3
    * `sgp1`: Singapore 1
    * `lon1`: London 1
    * `sfo1`: San Fransisco 1
    * `tor1`: Toronto 1
    * `fra1`: Frankfurt 1

*   `digitalocean_size`

    DigitalOcean droplet size.

    Supported size:

    * `512mb` (1 CPU)
    * `1gb` (1 CPU)
    * `2gb` (2 CPUs)
    * `4gb` (2 CPUs); this is the minimum recommended size hence it is set by default.
    * `8gb` (4 CPUs)
    * `16gb` (8 CPUs)
    * `32gb` (12 CPUs)
    * `48gb` (16 CPUs)
    * `64gb` (20 CPUs)

__Request example:__

```sh
curl http://localhost:8080/providers/digitalocean \
    -d name=my-do-provider \
    -d digitalocean_access_token=random-token \
    -d digitalocean_region=nyc3 \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/fe3eeb1d-7731-43f7-aa90-767d16fa3ab4

{
    "digitalocean_access_token": "random-token",
    "digitalocean_backups": false,
    "digitalocean_image": "ubuntu-14-04-x64",
    "digitalocean_ipv6": false,
    "digitalocean_private_networking": false,
    "digitalocean_region": "nyc3",
    "digitalocean_size": "4gb",
    "driver": "digitalocean",
    "id": "fe3eeb1d-7731-43f7-aa90-767d16fa3ab4",
    "name": "my-do-provider",
}
```

Please note, `driver` is also known as provider type.

__Status Code:__

* `201`: Provider is successfully created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

#### Generic Provider

    POST /providers/generic

__URL:__

    http://localhost:8080/providers/generic

__Form parameters:__

*   `name` (required)

    A unique name of the provider.

*   `generic_ip_address` (required)

    IP address of remote machine.

*   `generic_ssh_key` (required)

    Absolute path to private key used for SSH connection.

*   `generic_ssh_user` (required)

    SSH user used for SSH connection.

*   `generic_ssh_port` (required)

    Port used for SSH connection.

__Request example:__

```sh
curl http://localhost:8080/providers/digitalocean \
    -d name=my-gen-provider \
    -d generic_ip_address=172.10.10.10 \
    -d generic_ssh_key=/home/johndoe/.ssh/id_rsa \
    -d generic_ssh_user=root \
    -d generic_ssh_port=22 \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Content-Type: application/json
Location: http://localhost:8080/providers/17f9f346-0d28-45f1-96a0-49473cc21769

{
    "driver": "generic",
    "generic_ip_address": "172.10.10.10",
    "generic_ssh_key": "/home/johndoe/.ssh/id_rsa",
    "generic_ssh_port": 22,
    "generic_ssh_user": "root",
    "id": "17f9f346-0d28-45f1-96a0-49473cc21769",
    "name": "my-gen-provider",
}
```

Please note, `driver` is also known as provider type.

__Status Code:__

* `201`: Provider is successfully created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Get A Provider

    GET /providers/{id}

__URL:__

    http://localhost:8080/providers/{id}

__Request example:__

```sh
curl http://localhost:8080/providers/fe3eeb1d-7731-43f7-aa90-767d16fa3ab4 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "digitalocean_access_token": "random-token",
    "digitalocean_backups": false,
    "digitalocean_image": "ubuntu-14-04-x64",
    "digitalocean_ipv6": false,
    "digitalocean_private_networking": false,
    "digitalocean_region": "nyc3",
    "digitalocean_size": "4gb",
    "driver": "digitalocean",
    "id": "fe3eeb1d-7731-43f7-aa90-767d16fa3ab4",
    "name": "my-do-provider",
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
        "digitalocean_access_token": "random-token",
        "digitalocean_backups": false,
        "digitalocean_image": "ubuntu-14-04-x64",
        "digitalocean_ipv6": false,
        "digitalocean_private_networking": false,
        "digitalocean_region": "nyc3",
        "digitalocean_size": "4gb",
        "driver": "digitalocean",
        "id": "fe3eeb1d-7731-43f7-aa90-767d16fa3ab4",
        "name": "my-do-provider",
    },
    {
        "driver": "generic",
        "generic_ip_address": "172.10.10.10",
        "generic_ssh_key": "/home/johndoe/.ssh/id_rsa",
        "generic_ssh_port": 22,
        "generic_ssh_user": "root",
        "id": "17f9f346-0d28-45f1-96a0-49473cc21769",
        "name": "my-gen-provider",
    }
]
```

Note, if there's no provider, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Filter Providers

    GET /filter-providers

__URL:__

    http://localhost:8080/filter-providers

__Request example:__

```sh
curl http://localhost:8080/filter-providers/digitalocean -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "digitalocean_access_token": "random-token",
        "digitalocean_backups": false,
        "digitalocean_image": "ubuntu-14-04-x64",
        "digitalocean_ipv6": false,
        "digitalocean_private_networking": false,
        "digitalocean_region": "nyc3",
        "digitalocean_size": "4gb",
        "driver": "digitalocean",
        "id": "fe3eeb1d-7731-43f7-aa90-767d16fa3ab4",
        "name": "my-do-provider",
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
curl http://localhost:8080/providers/fe3eeb1d-7731-43f7-aa90-767d16fa3ab4 -X DELETE -i
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
