# Gluu Cluster Documentation

The Gluu Server cluster project is a docker-based recipe for deploying multiple instances of the Gluu Server to achieve an elastic, highly avalable, centralized authentication and authorization infrastructure.

Currently this product is in beta and is free to test. We still have some work to do before we are ready to ship the final version of gluu-master and gluu-consumer. For example, the logs that are currently written to `/tmp` need to be moved to `/var`. Also, there is no startup script for `gluuapi`.

Once finished, we anticipate pricing will be $750 USD/year per gluu-consumer. The gluu-master will remain free so that no license is required to evaluate the software. Only when you add servers 2-n will the cluster require a license.

Community support can be enlisted on the [Gluu website](http://support.gluu.org). Please use [Github](http://github.com/GluuFederation) to report bugs or request feature enhancements. Gluu also offers [VIP support](http://gluu.org/pricing) and can refer your organization to one of our world-class [integration partners](http://gluu.org/current-partners) for any custom development and integration needs.

The main documentation is organized into the following sections:

- *[Admin Guide](#admin-guide)*
- *[Reference](#reference)*

# Admin Guide
- [Getting Started](./admin-guide/getting-started/index.md)
- [Installation](./admin-guide/installation/index.md)
- [Testing](./admin-guide/testing/index.md)
- [Tuning](./admin-guide/tuning/index.md)
- [Monitoring](./admin-guide/monitoring/index.md)
- [Overview](./admin-guide/overview/index.md)
- [Components](./admin-guide/components/index.md)
- [Web Interface](./admin-guide/webui/index.md)

# Reference
- [API](./reference/api/index.md)
  - [Node](./reference/api/node.md)
  - [Cluster](./reference/api/cluster.md)
  - [Provider](./reference/api/provider.md)
  - [License Credential](./reference/api/license_credential.md)
  - [License](./reference/api/license.md)
