## License Credential API

License credential represents an entity to manage credentials for license bought from Gluu Inc. These credentials are required to generate metadata from signed license retrieved from Gluu Inc. license server.

---

### Create New License Credential

`POST /license_credential`

__URL:__

`http://localhost:8080/license_credential`

Form parameters:

*   `public_key` (required)

    Public key given when buying a license code.

*   `public_password` (required)

    Public password given when buying a license code.

*   `license_password` (required)

    License password given when buying a license code.

*   `name` (required)

    Short and descriptive credential name.

__Request example:__

```sh
curl http://localhost:8080/license_credential \
    -X POST -i \
    -d public_key=your-public-key \
    -d public_password=your-public-password \
    -d license_password=your-license-password \
    -d name=testing
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Location: http://localhost:8080/license_credential/3bade490-defe-477d-8146-be0f621940ed
Content-Type: application/json

{
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

Note: the API doesn't check whether the credentials passed as request parameters are the same credentials given by license server.
Bad credentials will affect signed license's metadata.
Fortunately, there's an API to update the credentials. See [Update A License Credential](./#update-a-license-credential) below.

__Status Code:__

* `201`: License credential has been created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Get A License Credential

`GET /license_credential/{id}`

__URL:__

`http://localhost:8080/license_credential/{id}`

__Request example:__

```sh
curl -i http://localhost:8080/license_credential/3bade490-defe-477d-8146-be0f621940ed
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

__Status Code:__

* `200`: License credential is exist.
* `404`: License credential is not exist.
* `500`: The server having errors.

---

### List All License Credentials

`GET /license_credential`

__URL:__

`http://localhost:8080/license_credential`

__Request example:__

```sh
curl http://localhost:8080/license_credential -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "id": "3bade490-defe-477d-8146-be0f621940ed",
        "license_password": "your-license-password",
        "name": "testing",
        "public_key": "your-public-key",
        "public_password": "your-public-password"
    }
]
```

Note, if there's no license credential, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Update A License Credential

`PUT /license_credential/{id}`

__URL:__

`http://localhost:8080/license_credential/{id}`

Form parameters:

*   `public_key` (required)

    Public key given when buying a license code.

*   `public_password` (required)

    Public password given when buying a license code.

*   `license_password` (required)

    License password given when buying a license code.

*   `name` (required)

    Short and descriptive credential name.

__Request example:__

```sh
curl http://localhost:8080/license_credential/3bade490-defe-477d-8146-be0f621940ed \
    -X PUT -i \
    -d public_key=your-public-key \
    -d public_password=your-public-password \
    -d license_password=your-license-password \
    -d name=testing-2
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "id": "3bade490-defe-477d-8146-be0f621940ed",
    "license_password": "your-license-password",
    "name": "testing-2",
    "public_key": "your-public-key",
    "public_password": "your-public-password"
}
```

If license credential successfully updated, all related signed licenses' metadata will be regenerated.

__Status Code:__

* `200`: License credential has been updated.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Delete A License Credential

`DELETE /license_credential/{id}`

__URL:__

`http://localhost:8080/license_credential/{id}`

__Request example:__

```sh
curl http://localhost:8080/license_credential/3bade490-defe-477d-8146-be0f621940ed -X DELETE -i
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: License credential has been deleted.
* `404`: License credential is not exist.
* `500`: The server having errors.
