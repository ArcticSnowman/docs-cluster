[TOC]

# Installation
The Gluu Cluster only supports Ubuntu for now. we need two container to install gluu cluster management system.
1. gluuengine
2. gluuwebui

## Prerequisites

### Minimum Linux Kernel

Cluster requires at least kernel 3.10 at minimum. We can check whether we're using supported kernel.

    uname -r

Please note, due to [issue with kernel 3.13.0-77](../known-issues#unsupported-kernel), this version should be avoided.

## Docker Engine

### Installing Docker Engine

Run this:

```
$ sudo curl -fsSL https://raw.githubusercontent.com/GluuFederation/cluster-tools/master/get_docker.sh | sh
```

## Installing and running gluu cluster management system

### Installing gluu-engine and gluu-webui image:

From docker hub (not yet released):

```sh
docker pull gluu/gluuengine
docker pull gluu/gluuwebui
```

Alternatively you can build it:

```sh
git clone https://github.com/GluuFederation/gluu-docker.git
docker build --rm=true --force-rm=true --tag=gluuengine gluu-docker/ubuntu/14.04/gluuengine
docker build --rm=true --force-rm=true --tag=gluuengine gluu-docker/ubuntu/14.04/gluuwebui
```

### Preparing Database

All data is saved into MongoDB, hence we need to prepare the database before running `gluu-engine`:

```sh
docker run -d --name mongo -v /var/lib/gluuengine/db/mongo:/data/db mongo
```

### Running gluu-engine

We need mongodb for gluuengine:

```sh
docker run -d --name mongo -v /var/lib/gluuengine/db/mongo:/data/db mongo
```

Running gluu-engine:

```sh
docker run -d -p 127.0.0.1:8080:8080 --name gluuengine \
    -v /var/log/gluuengine:/var/log/gluuengine \
    -v /var/lib/gluuengine:/var/lib/gluuengine \
    -v /var/lib/gluuengine/machine:/root/.docker/machine \
    --link mongo:mongo gluuengine
```

### Running gluu-webui

```sh
docker run -d -p 127.0.0.1:8800:8800 --name gluuwebui \
    --link gluuengine:gluuengine gluuwebui
```

now open your browser and hit, http://localhost:8800

There are few things we need to know about Gluu Cluster Web UI:

1. The application is bind to 127.0.0.1 (localhost) as currently there's no protection mechanism yet.
2. The application is listening on port 8800 to avoid conflict with port used by nodes.
