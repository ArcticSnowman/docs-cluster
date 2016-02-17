# Welcome to the Gluu Enterprise Edition (EE) Documentation

Gluu EE is a high availability and high reliability solution for enterprise authentication and authorization. Gluu EE includes the same components as [Community Edition (CE)](http://gluu.org/docs), but instead of using `chroot` to deliver all the components in one package, each component in EE is delivered in its own `docker` contaner. In addition, EE includes a management system, called Gluu Flask, which enables rapid elasticity and scalability across multiple cloud providers in multiple geographic regions. 

The Gluu Server EE is divided into two packages identified as "master" and "consumer". The "master" package is offered free and the "consumer" package requires a commercial license. For pricing information, please [schedule a meeting with us](http://gluu.org/booking).

**Note:** currently Enterprise Edition includes the Shibboleth SAML IDP, oxAuth OpenID Connect IDP and UMA Authorization Server (AS), oxTrust web GUI, and LDAP. 

# Admin Guide
- [Overview](./admin-guide/overview/index.md)
- [Getting Started](./admin-guide/getting-started/index.md)
- [Installation](./admin-guide/installation/index.md)
- [Cluster Management](./admin-guide/cluster-management/index.md)
- [Components](./admin-guide/components/index.md)
- [Web Interface](./admin-guide/webui/index.md)
- [Troubleshooting](./admin-guide/troubleshooting/index.md)
- [Recovery](./admin-guide/recovery/index.md)
- [Migration](./admin-guide/migration/index.md)

# Reference
- [API](./reference/api/index.md)
  - [Cluster](./reference/api/cluster.md)
  - [Provider](./reference/api/provider.md)
  - [Node](./reference/api/node.md)
  - [License Key](./reference/api/license_key.md)
