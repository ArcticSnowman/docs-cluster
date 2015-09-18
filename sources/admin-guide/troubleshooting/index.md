# Troubleshooting
[TOC]

## Loggin In Web Interface
To log into the web interface, it is necessary to ssh into the master provider, as the interface is run locally and it is not facing the internet for security reasons.

Run the following command to SSH for accessing the web interface:

```
ssh -L <port-name>:localhost:8800 -i <cluster-name> ubuntu@<cluster-domain>
```

Point your browser to the following address to access the webui:

`http://localhost:<port-name>`

Note: use the same portname for both the commands.
## Testing the Cluster
This section will be added soon.

## Log Files
### Setup Logs
The setup logs for the cluster nodes are available `/var/log/gluu/` of the master container. The following path shows the exact location of the logs. The term `<node-name>` represents the four nodes that are LDAP, oxAuth, oxTrust and Apache.

`/var/log/gluu/<node-name>-setup.log`

## Recovering Cluster
There are instances when the cluster server may be rebooted, or for some unavoidable circumstances, the cluster server was shutdown and booted again. The following shows how to recover the master and consumer cluster after such an incident.
It is necessaty to log into the web interface or into the master container before the cluster can be recovered. Follow the instructions given above to log into the web interface, and then procced with the following sections.

### Recovering Provider ID
It is a must that the master provider/web interface is logged into before continuting any further. Please see the instuctions aove to log into the web interface. The provider ID is necessary to recover gluu-master and gluu-consumer. The following instructions outline the methods to get the provider id:
Run the following command to SSH for accessing the web interface:
```
 ssh -L <port-name>:localhost:8800 -i <cluster-name> ubuntu@<cluster-domain>
```
From here there are two ways to find the provider ID:
1. Point your browser to the following address to access the webui and click on provider:
`http://localhost:<port-name>`

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

