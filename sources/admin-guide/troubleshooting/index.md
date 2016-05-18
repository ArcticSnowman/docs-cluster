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

The term `<container-name>` represents the currently-supported containers that are `ldap`, `oxauth`, `oxtrust`, `oxidp`, and `nginx`.

## Recovering Cluster
There are instances when the cluster server may be rebooted, or for some unavoidable circumstances, the cluster server was shutdown and booted again.

For a detailed step by step cluster recovery see the [Recovery Page](../recovery/).

### Not Enough Entropy

In some virtual machines hosted at cloud providers, oxAuth/oxTrust/oxIdp may have a slow restart due to insufficient entropy.
To check available entropy, we can use the following command:

    cat /proc/sys/kernel/random/entropy_avail

If the number returned from the command is above 1000, we can __skip__ this section.
Otherwise, we need to feed the entropy with other tools.

One of the tools that we can use is `rng-tools`. It's available from Ubuntu repository:

    apt-get install rng-tools

We probably will encounter this error after installing `rng-tools`.

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

Afterwards, restart the `rng-tools` service:

    service rng-tools restart

After `rng-tools` service is running successfully, we can check the available entropy again.

    cat /proc/sys/kernel/random/entropy_avail

The number returned from the command above is around 3000.
From this point, we can say that we have enough entropy required
by oxAuth/oxTrust/oxIdp to start properly.
