# Welcome to the Gluu Server Docker Edition (DE) Documentation

The Gluu Server Docker Edition ([DE][de]) is a high availability and high reliability solution for enterprise authentication and authorization. Gluu [DE][de] includes the same components as [Community Edition (CE)](http://gluu.org/docs), but instead of using `chroot` to deliver all the components in one package, each component in [DE][de] is delivered in its own `docker` container. In addition, [DE][de] includes a management system, called Gluu Flask, which enables rapid elasticity and scalability of individual containers across multiple cloud providers in multiple geographic regions.

The Gluu Server [DE][de] is divided into two packages identified as "master" and "consumer". The "master" package is offered free and the "consumer" package requires a commercial license. For pricing information, please [schedule a meeting with us](http://gluu.org/booking).

**Note:** currently [DE][de] includes the Shibboleth SAML IDP, oxAuth OpenID Connect IDP and UMA Authorization Server (AS), oxTrust web GUI, and LDAP.

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
- [Known Issues](./admin-guide/known-issues/index.md)

# Reference
- [API](./reference/api/index.md)
  - [Cluster](./reference/api/cluster.md)
  - [Provider](./reference/api/provider.md)
  - [Node](./reference/api/node.md)
  - [License Key](./reference/api/license_key.md)
  - [Node Log](./reference/api/node_log.md)

[de]: http://docker.com "Docker Edition"
