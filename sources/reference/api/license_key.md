## License Key API

License key represents an entity to manage keys for license bought from Gluu Inc. These keys are required to generate metadata from signed license retrieved from Gluu Inc. license server.

---

### Create New License Key

`POST /license_keys`

__URL:__

`http://localhost:8080/license_keys`

Form parameters:

*   `public_key` (required)

    Public key given when buying a license code.

*   `public_password` (required)

    Public password given when buying a license code.

*   `license_password` (required)

    License password given when buying a license code.

*   `name` (required)

    Short and descriptive key name.

*   `code` (required)

    License code.

__Request example:__

```sh
curl http://localhost:8080/license_keys \
    -d public_key=your-public-key \
    -d public_password=your-public-password \
    -d license_password=your-license-password \
    -d name=testing \
    -d code=your-code \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Location: http://localhost:8080/license_keys/3bade490-defe-477d-8146-be0f621940ed
Content-Type: application/json

{
    "code": "your-code",
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

Note: the API doesn't check whether the keys passed as request parameters are the same keys given by license server.
Bad keys will affect signed license's metadata.
Fortunately, there's an API to update the keys. See [Update A License key](./#update-a-license-key) below.

__Status Code:__

* `201`: License key has been created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `403`: Access denied.
* `500`: The server having errors.

---

### Get A License key

`GET /license_keys/{id}`

__URL:__

`http://localhost:8080/license_keys/{id}`

__Request example:__

```sh
curl http://localhost:8080/license_keys/3bade490-defe-477d-8146-be0f621940ed -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "code": "your-code",
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

__Status Code:__

* `200`: License key is exist.
* `404`: License key is not exist.
* `500`: The server having errors.

---

### List All License Keys

`GET /license_keys`

__URL:__

`http://localhost:8080/license_keys`

__Request example:__

```sh
curl http://localhost:8080/license_keys -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "code": "your-code",
        "id": "3bade490-defe-477d-8146-be0f621940ed",
        "license_password": "your-license-password",
        "name": "testing",
        "public_key": "your-public-key",
        "public_password": "your-public-password"
    }
]
```

Note, if there's no license key, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Update A License Key

`PUT /license_keys/{id}`

__URL:__

`http://localhost:8080/license_keys/{id}`

Form parameters:

*   `public_key` (required)

    Public key given when buying a license code.

*   `public_password` (required)

    Public password given when buying a license code.

*   `license_password` (required)

    License password given when buying a license code.

*   `name` (required)

    Short and descriptive key name.

*   `code` (required)

    License code.

__Request example:__

```sh
curl http://localhost:8080/license_keys/3bade490-defe-477d-8146-be0f621940ed \
    -d public_key=your-public-key \
    -d public_password=your-public-password \
    -d license_password=your-license-password \
    -d name=testing-2 \
    -d code=your-code \
    -X PUT -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "code": "your-code",
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing-2",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

If license key successfully updated, all related signed licenses' metadata will be regenerated.

__Status Code:__

* `200`: License key has been updated.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Delete A License Key

`DELETE /license_keys/{id}`

__URL:__

`http://localhost:8080/license_keys/{id}`

__Request example:__

```sh
curl http://localhost:8080/license_keys/3bade490-defe-477d-8146-be0f621940ed -X DELETE -i
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: License key has been deleted.
* `404`: License key is not exist.
* `500`: The server having errors.
