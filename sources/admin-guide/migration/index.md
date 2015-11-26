# Migration

[TOC]

## Version 0.4.1-8

1.  Install latest `gluu-flask` package updates:

        apt-get update && apt-get install -y gluu-flask

2.  Remove existing ldap, oxauth, oxtrust, oxidp nodes.
3.  Remove gluuopendj, gluuoxtrust, gluuoxidp, and gluuoxauth images:

        docker rmi gluuopendj gluuoxtrust gluuoxidp gluuoxauth

4.  Re-deploy ldap, oxauth, oxidp (optional), and oxtrust nodes.

Changelog for v0.4.1 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-041).

## Version 0.4.1

1.  For master provider, update `gluu-master`, `gluu-flask`, and `gluu-agent` packages.

        apt-get update && apt-get install -y gluu-master gluu-flask gluu-agent

    Note: since newer `gluu-master` package introduces docker v1.8.3, existing nodes might be
    crashed. To recover the nodes, run the recovery command:

        service gluu-agent recover

2.  For consumer provider, update `gluu-consumer` and `gluu-agent` packages.

        apt-get update && apt-get install -y gluu-consumer gluu-agent

    Note: since newer `gluu-consumer` package introduces docker v1.8.3, existing nodes might be
    crashed. To recover the nodes, run the recovery command:

        service gluu-agent recover

Changelog for v0.4.1 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-041).

## Version 0.4.0

Steps to migrate to Gluu Cluster v0.4.0:

1.  Update `gluu-flask`, `gluu-agent` and `weave` packages:

        apt-get update && apt-get install -y gluu-flask gluu-agent weave

2.  Remove old weave container:

        docker rm -f weave

3.  Update weave image:

        weave setup

4.  Remove existing ldap, oxauth, httpd and oxtrust nodes (containers).

5.  Remove gluuopendj, gluuoxtrust, and gluuoxauth images:

        docker rmi gluuopendj gluuoxtrust gluuoxauth

6.  Re-deploy ldap, oxauth, oxidp (optional), nginx, and oxtrust nodes.

Changelog for v0.4.0 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-040).

## Version 0.3.3-12

Steps to migrate to Gluu Cluster v0.3.3-12:

1.  Update `gluu-flask` via `apt-get update && apt-get install -y gluu-flask`.
2.  Remove ldap, oxauth, httpd and oxtrust nodes (containers).
3.  Remove gluuoxauth and gluuoxtrust images via `docker rmi` command.
4.  Re-deploy ldap, oxauth, httpd, and oxtrust nodes.
5.  Install gluu-agent: `apt-get update && apt-get install gluu-agent`.

Changelog for v0.3.3 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-033).
