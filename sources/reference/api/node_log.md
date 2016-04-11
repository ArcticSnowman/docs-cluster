## Node Log API

[TOC]

Node Log is an entity holds setup and teardown logs for specific node.

---

### Get A Node Log

    GET /node_logs/{id}

__URL:__

    http://localhost:8080/node_logs/{id}

__Request example:__

```sh
curl http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "id": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "node_name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "setup_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/setup",
    "state": "SETUP_FINISHED",
    "teardown_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/teardown"
}
```

Note: `teardown_log_url` may have empty string value if node is not deleted yet.

__Status Code:__

* `200`: Node Log is exist.
* `404`: Node Log is not exist.
* `500`: The server having errors.

---

### Get A Node Setup Log

    GET /node_logs/{id}/setup

__URL:__

    http://localhost:8080/node_logs/{id}/setup

__Request example:__

```sh
curl http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/setup -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

```

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "id": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "node_name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "setup_log_contents": [
        "2016-02-23 00:48:47,052 - gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 - INFO  - Waiting for minion to connect; sleeping for 10 seconds",
        "2016-02-23 00:48:57,058 - gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 - INFO  - Preparing remote execution; sleeping for 15 seconds",
        "2016-02-23 00:49:12,074 - gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 - INFO  - attaching weave IP address 10.2.0.1/16",
        "2016-02-23 00:53:17,153 - gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 - INFO  - gluuopendj setup is finished (242.836978912 seconds)"
    ],
    "setup_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/setup",
    "state": "SETUP_FINISHED",
    "teardown_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/teardown"
}
```

__Status Code:__

* `200`: Node Log is exist.
* `404`: Node Log is not exist.
* `500`: The server having errors.

---

### Get A Node Teardown Log

    GET /node_logs/{id}/teardown

__URL:__

    http://localhost:8080/node_logs/{id}/teardown

__Request example:__

```sh
curl http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/teardown -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "id": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "node_name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
    "setup_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/setup",
    "state": "TEARDOWN_FINISHED",
    "teardown_log_contents": [
        "2016-02-23 00:53:17,153 - gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 - INFO  - gluuopendj teardown is finished (20.836978912 seconds)"
    ],
    "teardown_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/teardown"
}
```


__Status Code:__

* `200`: Node Log is exist.
* `404`: Node Log is not exist.
* `500`: The server having errors.

---

### List All Node Logs

    GET /node_logs

__URL:__

    http://localhost:8080/node_logs

__Request example:__

```sh
curl http://localhost:8080/node_logs -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "id": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
        "node_name": "gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043",
        "setup_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/setup",
        "state": "SETUP_FINISHED",
        "teardown_log_url": "http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043/teardown"
    }
]
```

Note:

* If there's no node logs, the response body will return empty list.
* `teardown_log_url` may have empty string value if node is not deleted yet.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A Node Log

    DELETE /node_logs/{id}

__URL:__

    http://localhost:8080/node_logs/{id}

```sh
curl http://localhost:8080/node_logs/gluuopendj_f42dd3bf-28c8-450c-b221-77b677b59043 -X DELETE -i --max-time 300
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: Node Log has been deleted.
* `404`: Node Log is not exist.
* `500`: The server having errors.
