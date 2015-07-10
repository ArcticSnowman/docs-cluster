## License API

License represents an entity to manage signed license retrieved from Gluu Inc. license server.

---

### Create New License

`POST /licenses`

Note: to create a license entity, license key must be created first.
See [Create New License Key](../license_key/#create-new-license-key) section for details.

__URL:__

`http://localhost:8080/licenses`

Form parameters:

*   `license_key_id` (required)

    ID of license key.

__Request example:__

```sh
curl http://localhost:8080/licenses \
    -d license_key_id=3bade490-defe-477d-8146-be0f621940ed \
    -X POST -i
```

__Response example:__

```http
HTTP/1.0 201 CREATED
Location: http://localhost:8080/licenses/1186482d-fafd-4a97-be7f-d4f3b4167e88
Content-Type: application/json

{
    "code": "3bade490-defe-477d-8146-be0f621940ec",
    "license_key_id": "3bade490-defe-477d-8146-be0f621940ed",
    "expired": false,
    "id": "1186482d-fafd-4a97-be7f-d4f3b4167e88",
    "metadata": {
        "expiration_date": 1465759806000,
        "license_count_limit": 30,
        "license_features": [
            "gluu_server"
        ],
        "license_name": "testing",
        "license_type": null,
        "multi_server": false,
        "thread_count": 3
    },
    "valid": true
}
```

`metadata` and `valid` key pairs are generated based on license key.
If license uses incorrect license key, the response would have a different result.

```http
HTTP/1.0 201 CREATED
Location: http://localhost:8080/licenses/1186482d-fafd-4a97-be7f-d4f3b4167e88
Content-Type: application/json

{
    "code": "3bade490-defe-477d-8146-be0f621940ec",
    "license_key_id": "3bade490-defe-477d-8146-be0f621940ed",
    "expired": true,
    "id": "1186482d-fafd-4a97-be7f-d4f3b4167e88",
    "metadata": {},
    "valid": false
}
```

To fix this problem, update the related license key.
See [Update A License Key](../license_key/#update-a-license-key) section for details.

__Status Code:__

* `201`: License has been created.
* `400`: Bad request. Possibly malformed/incorrect parameter value.
* `500`: The server having errors.

---

### Get A License

`GET /licenses/{id}`

__URL:__

`http://localhost:8080/licenses/{id}`

__Request example:__

```sh
curl http://localhost:8080/licenses/1186482d-fafd-4a97-be7f-d4f3b4167e88 -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

{
    "code": "3bade490-defe-477d-8146-be0f621940ec",
    "license_key_id": "3bade490-defe-477d-8146-be0f621940ed",
    "expired": false,
    "id": "1186482d-fafd-4a97-be7f-d4f3b4167e88",
    "metadata": {
        "expiration_date": 1465759806000,
        "license_count_limit": 30,
        "license_features": [
            "gluu_server"
        ],
        "license_name": "testing",
        "license_type": null,
        "multi_server": false,
        "thread_count": 3
    },
    "valid": true
}
```

__Status Code:__

* `200`: License is exist.
* `404`: License is not exist.
* `422`: Server unable to retrieve signed license from license server.
* `500`: The server having errors.

---

### List All License

`GET /licenses`

__URL:__

`http://localhost:8080/licenses`

__Request example:__

```sh
curl http://localhost:8080/licenses -i
```

__Response example:__

```http
HTTP/1.0 200 OK
Content-Type: application/json

[
    {
        "code": "3bade490-defe-477d-8146-be0f621940ec",
        "license_key_id": "3bade490-defe-477d-8146-be0f621940ed",
        "expired": false,
        "id": "1186482d-fafd-4a97-be7f-d4f3b4167e88",
        "metadata": {
            "expiration_date": 1465759806000,
            "license_count_limit": 30,
            "license_features": [
                "gluu_server"
            ],
            "license_name": "testing",
            "license_type": null,
            "multi_server": false,
            "thread_count": 3
        },
        "valid": true
    }
]
```

Note, if there's no license, the response body will return empty list.

__Status Code:__

* `200`: Request is succeed.
* `500`: The server having errors.

---

### Delete A License

`DELETE /licenses/{id}`

__URL:__

`http://localhost:8080/licenses/{id}`

__Request example:__

```sh
curl http://localhost:8080/licenses/1186482d-fafd-4a97-be7f-d4f3b4167e88 -X DELETE -i
```

__Response example:__

```http
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
```

__Status Code:__

* `204`: License has been deleted.
* `404`: License is not exist.
* `500`: The server having errors.
