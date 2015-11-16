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

    We will probably see the following error:

        Starting Hardware RNG entropy gatherer daemon: (failed).
        invoke-rc.d: initscript rng-tools, action "start" failed.

    We can fix it by modifying HRNGDEVICE in `/etc/default/rng-tools`.

        # Configuration for the rng-tools initscript
        # $Id: rng-tools.default,v 1.1.2.5 2008-06-10 19:51:37 hmh Exp $

        # This is a POSIX shell fragment

        # Set to the input source for random data, leave undefined
        # for the initscript to attempt auto-detection.  Set to /dev/null
        # for the viapadlock driver.
        #HRNGDEVICE=/dev/hwrng
        #HRNGDEVICE=/dev/null
        HRNGDEVICE=/dev/urandom

    Afterwards, restart the rng-tools service:

        service rng-tools restart

    After rng-tools service is running successfully, we can check the available entropy again.

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


## weave is Not Launched After Registering Provider

After registering a provider, we may wondering why weave is not launched.
We can check whether weave is launched by using the command below:

    docker ps | grep weave

If the command above doesn't show anything related to weave, there could be
a problem with salt communication between master and minion
(since weave is launched via salt remote execution).

1.  Ensure salt-master is running:

        service salt-master status

    If salt-master is not running, make it so:

        service salt-master restart

2.  Check minion's key is registered in salt-key:

        salt-key

    Note, in this context, we use `gluu.example.com` as the minion's key.

    If minion's key is not accepted, we may need to check whether we use a correct
    `hostname` value when registering the provider.

3.  If minion's key is accepted, try to ping the minion:

        salt gluu.example.com test.ping

    If the command above returns nothing, check salt-minion service:

        service salt-minion status

    If salt-minion service is stopped, check the minion log. But first things first,
    we need to enable debugging mode temporarily and restart the service:

        echo 'log_level: debug' >> /etc/salt/minion
        service salt-minion restart

    Afterwards, check the minion's log:

        tail /var/log/salt/minion

    From the log, we can see errors when running minion, and depends on the usecase, we can find a solution on
    how to solve the issue.

    Please note, running minion with `log_level` set to debug may slowing down the minion
    process. It's recommended to remove the `log_level` line in `/etc/salt/minion` after the issue has been solved.

### Usecase #1: Invalid Master Public Key

Below is an example of minion's log contents:

```
The master key has changed, the salt master could have been subverted, verify salt master's public key
2015-10-27 21:45:33,643 [salt.crypt       ][CRITICAL] The Salt Master server's public key did not authenticate!
The master may need to be updated if it is a version of Salt lower than 2014.7.4, or
If you are confident that you are connecting to a valid Salt Master, then remove the master public key and restart the Salt Minion.
The master public key can be found at:
/etc/salt/pki/minion/minion_master.pub
```

From the example above, we know that minion is unable to authenticate to master due to invalid master public key.
The last lines also told us to remove `/etc/salt/pki/minion/minion_master.pub` and restart the salt-minion service.

    rm /etc/salt/pki/minion/minion_master.pub
    service salt-minion restart

Once `/etc/salt/pki/minion/minion_master.pub` has been removed and salt-minion has been restarted, we can try ping the minion again:

    salt gluu.example.com test.ping

A successfull ping request returns the output below:

    gluu.example.com:
        True

The communication between salt-master and salt-minion is now established, hence we can launch weave again by updating the provider
via web UI or [direct API](../../reference/api/provider.md#update-a-provider) request.
After the provider is updated, wait for few seconds and check whether weave is successfully launched or not:

    docker ps

If weave container is listed on the output, we have successfully launching weave.

```
CONTAINER ID        IMAGE                      COMMAND                STATUS              NAMES
fd4d3cd4bf70        weaveworks/weave:v0.10.0   "/home/weave/weaver    Up About a minute   weave
```
