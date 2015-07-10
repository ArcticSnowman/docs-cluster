# Gluu Cluster Documentation

If you want to deploy multiple replicated Gluu Servers to enable a highly available access management service,
you're in the right place!

The Gluu Server has two distributions: (1) Community Edition--where all Gluu services are deployed in one `chroot`
container; (2) HA Edition--where each service is deployed in its own `docker` container. The HA edition has a "master" 
package and a "consumer" package. Licenses are required for each consumer. 

`Note`: currently the HA Edition only supports oxAuth, oxTrust and LDAP--not the Shibboleth SAML IDP (yet!).

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
- [Monitoring](./admin-guide/monitoring/index.md)
- [Overview](./admin-guide/overview/index.md)
- [Components](./admin-guide/components/index.md)
- [Web Interface](./admin-guide/webui/index.md)

# Reference
- [API](./reference/api/index.md)
  - [Node](./reference/api/node.md)
  - [Cluster](./reference/api/cluster.md)
  - [Provider](./reference/api/provider.md)
  - [License Key](./reference/api/license_key.md)
  - [License](./reference/api/license.md)
