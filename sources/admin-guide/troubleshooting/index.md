# Troubleshooting
[TOC]

## Testing the Cluster
This section will be added soon.

## Log Files
### Setup Logs
The setup logs for the cluster nodes are available `/var/log/gluu/` of the master container. The following path shows the exact location of the logs. The term `<node-name>` represents the four nodes that are LDAP, oxAuth, oxTrust and Apache.

`/var/log/gluu/<node-name>-setup.log`

## Recovering Cluster
There are instances when the cluster server may be rebooted, or for some unavoidable circumstances, the cluster server was shutdown and booted again. The following shows how to recover the master and consumer cluster after such an incident.
It is necessaty to log into the web interface or into the master container before the cluster can be recovered.

The following provides a simple recovery method. However, gluu cluster is also shipped with a recovery script. 

* [Cluster recovery with recovery script](http://www.gluu.org/docs-cluster/admin-guide/recovery/)
### Recovering Provider ID
It is a must that the master provider/web interface is logged into before continuting any further. Please see the instuctions aove to log into the web interface. The provider ID is necessary to recover gluu-master and gluu-consumer. There are two ways to get the provider ID.

1. [Log into the web interface](http://www.gluu.org/docs-cluster/admin-guide/webui/) and click on provider.

2. Use curl command:
```
curl -i http://localhost:8080/providers
```

The response will contain the provider id in the string like this:
` "type": "consumer", "id": "0b42af6b-ac61-4c50-92bd-28553db48e7c`

### Recovering Master
To recover the gluu master, run the command:

`API_ENV=prod SALT_MASTER_IPADDR=$master_ip_address gluuapi recover $master_provider_id`

### Recovering Consumer
To recover the gluu consumer, run the following command:

`API_ENV=prod SALT_MASTER_IPADDR=$master_ip_address gluuapi recover $consumer_provider_id`

### More Info

For a detailed step by step cluster recovery see the [Recovery Page](http://www.gluu.org/docs-cluster/admin-guide/recovery/).
