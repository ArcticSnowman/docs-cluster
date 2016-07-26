# Troubleshooting

[TOC]

## Testing the Cluster
This section will be added soon.

## Log Files
The setup and deployment logs for the cluster container are available via Container Log API.

Example:

    # setup log
    curl localhost:8080/container_logs/<container-name>/setup

    # teardown log
    curl localhost:8080/container_logs/<container-name>/teardown

The term `<container-name>` represents the currently-supported containers that are `ldap`, `oxauth`, `oxtrust`, and `nginx`.

## Recovering Cluster
There are instances when the cluster server may be rebooted, or for some unavoidable circumstances, the cluster server was shutdown and booted again.

For a detailed step by step cluster recovery see the [Recovery Page](../recovery/).
