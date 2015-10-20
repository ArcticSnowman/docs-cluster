# Recovery Overview
[TOC]

A provider may crashes due to various reasons (i.e. power outage).
When it crashes, all containers (weave, prometheus, and nodes) will crash as well.

As of v0.3.3-12, recovery script is moved to Gluu Agent package.
Refer to installation section to install the package for [master](../installation/#installing-gluu-agent-on-master-provider) and [consumer](../installation/#installing-gluu-agent-on-consumer-provider) provider.

One of Gluu Agent jobs is to recover nodes automatically after provider is rebooted.
We can also recover the provider manually using the following command:

    service gluu-agent recover

The recovery process is logged to `/var/log/gluuagent-recover.log` so we can investigate the result.

Note, Gluu Agent relies on local cluster data that is pushed by `gluu-flask` v0.3.3-12 to each provider,
hence it is recommended to upgrade to `gluu-flask` v0.3.3-12.
