# Troubleshooting
[TOC]

## Testing the Cluster
This section will be added soon.

## Log Files
### Setup Logs
The setup logs for the cluster nodes are available `/var/log/gluu/` of the master container. The following path shows the exact location of the logs. The term `<node-name>` represents the four nodes that are LDAP, oxAuth, oxTrust and Apache.

`/var/log/gluu/<node-name>-setup.log`

## Recovering Cluster
There are instances when the cluster server may be rebooted, or for some unavoidable circumstances, the cluster server was shutdown and booted again.

Starting from v0.3.3-12, recovery script is moved to a package called Gluu Agent. For a detailed step by step cluster recovery see the [Recovery Page](../recovery/).

## Slow oxAuth or oxTrust Startup

In some virtual machines hosted at cloud providers, oxAuth/oxTrust may have a slow start due to insufficient entropy. To check whether oxAuth or oxTrust in our cluster is affected by this issue, we can do the following steps:

1.  After deploying oxAuth/oxTrust, login to the container:

        docker exec -ti $node_id bash

2.  Check if port 8005 is listening.

        netstat -napt | grep 8005

    If the command returns empty output, wait for 5 minutes,
    then repeat the command above.
    If the port is not ready after 5 minutes, we can check Tomcat log:

        cat /opt/tomcat/logs/localhost.*.log

    If log file is empty, probably we have issue with insufficient entropy. Now we need to check something in provider itself. We can safely
    logout from container.

3.  Check the available entropy:

        cat /proc/sys/kernel/random/entropy_avail

    If the number returned from the command above is below 3000,
    we need to feed the entropy with other tools.

    One of tool we can use is `rng-tools`. It's available from Ubuntu
    repository:

        apt-get install rng-tools

    rng-tools will run as a service. After the package is already installed, we can check the available entropy again.

        cat /proc/sys/kernel/random/entropy_avail

    The number returned from the command above is around 3000.
    From this point, we can say that we have enough entropy required
    by oxAuth/oxTrust to start properly.

4.  Restart oxAuth or oxTrust:

        docker stop $node_id

    Note, we need to use Gluu Agent to bring back the node properly.
    Make sure we have installed Gluu Agent. If we don't have the package yet, it's simply install the package from the repositories.

        apt-get install gluu-agent

    Afterwards, run the recovery command manually:

        service gluu-agent recover

    To check whether we have fixed the slow startup issue, repeat step 1-2.
