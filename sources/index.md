# Gluu Cluster Documentation

If you want to deploy multiple replicated Gluu Servers to enable a highly available access management service,
you're in the right place!

The Gluu Server has two distributions: (1) Community Edition--where all Gluu services are deployed in one `chroot`
container; (2) Cluster Edition--where each service is deployed in its own `docker` container. The Cluster Edition has a
"master" package and a "consumer" package. The master package is free, and the consumer package requires a commercial
license.

`Note`: currently the Cluster Edition only supports oxAuth, oxTrust and LDAP. Support for Shibboleth IDP version 3
is planed for release 0.4 of the Gluu Clusters packages.

Community support can be enlisted on the [Gluu website](http://support.gluu.org). Please use
[Github](http://github.com/GluuFederation) to report bugs or request feature enhancements. Gluu also offers
[VIP support](http://gluu.org/pricing) and can refer your organization to one of our world-class
[integration partners](http://gluu.org/current-partners) for any custom development and integration needs.

The main documentation is organized into the following sections:

- *[Admin Guide](#admin-guide)*
- *[Reference](#reference)*

# Admin Guide
- [Getting Started](./admin-guide/getting-started/index.md)
- [Installation](./admin-guide/installation/index.md)
- [Testing](./admin-guide/testing/index.md)
- [Tuning](./admin-guide/tuning/index.md)
- [Overview](./admin-guide/overview/index.md)
- [Components](./admin-guide/components/index.md)
- [Web Interface](./admin-guide/webui/index.md)
- [Troubleshooting](./admin-guide/troubleshooting/index.md)

# Reference
- [API](./reference/api/index.md)
  - [Node](./reference/api/node.md)
  - [Cluster](./reference/api/cluster.md)
  - [Provider](./reference/api/provider.md)
  - [License Key](./reference/api/license_key.md)
