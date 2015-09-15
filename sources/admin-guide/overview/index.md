# Overview

[TOC]

Gluu Cluster promises scalability, reliability and fail-over mechanism through its brilliant design implemented using [Docker](https://www.docker.com/).
The cluster server also comes with [DOSarrest](http://www.dosarrest.com/), enabling protection from distrubuted denial of service attacks.
The cluster server deploys the Gluu IdP, the identity management suite, capable of authenticating and authorizing using both SAML and OpenID Connect protocol.
The single deployment of Gluu Server is recommended to try before the Cluster as it requires a commercial license to deploy a cluster in multiple locations. 
The overview of the Gluu Server is available [here](http://www.gluu.org/docs/admin-guide/getting-started/).

## Design

The Gluu Cluster has two principle compoents that are interdependent, the master and the consumer.
The master package must be installed for the cluster to be functional, and without the master package, the consumer will not work.
The installation instructions are available in the [Installation Docs](http://www.gluu.org/docs-cluster/admin-guide/getting-started/).


## Functionalities

The core functionalities of the Gluu Server is available in the cluster design with a few additional features such as cluster monitoring, DDOS protection and a fail-safe system.
The Cluster monitoring system uses Promethius for alerts and reports on the nodes in real-time.
The cluster administrator can use the dashboard to check on the health and activity, view logs and other Gluu Server administration tasks. The Gluu Server Administration Guide is available [here](http://www.gluu.org/docs/admin-guide/introduction/).

## Components

The Gluu Cluster takes advantage of the latest free, open-source, components which work togather to provide a hickup-free cluster environment.
The components are listed in the [components page](http://127.0.0.1:8000/admin-guide/components/#components).

## License
There are 3 available licenses for Gluu Cluster deployment:

1. E-Commerce: This license allows a cluster deployed in two separate locations or virtual machines from two different vendors. An example would be a cluster with its master package deployed in Amazon and the consumer deployed in Rackspace. For more information on pricing please go to [Gluu Pricing](http://www.gluu.org/gluu-server/pricing/).

2. Premium: This package allows the cluster to be deployed to five (5) locations or virtual machines. The premium package also includes Premium Support. The details are given in the [Gluu Pricing](http://www.gluu.org/gluu-server/pricing/) page.

3. Enterprise: The Enterprise package includes a license for unlimited cluster deployment with an enterprise support. This package will allow any enterprise to create any number of clusters for development, QA and production.
