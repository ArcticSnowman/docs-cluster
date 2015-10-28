# Migration

[TOC]

## Version 0.4.0-1

Steps to migrate to Gluu Cluster v0.4.0:

1.  Update `gluu-flask` and `weave` packages:

        apt-get update && apt-get install -y gluu-flask weave

2.  Remove old weave container:

        docker rm -f weave

3.  Update weave image:

        weave setup

4.  Update all providers via web UI or Provider API request directly.

5.  Remove existing ldap, oxauth, httpd and oxtrust nodes (containers).

6.  Remove gluuoxtrust image:

        docker rmi gluuoxtrust

7.  Re-deploy ldap, oxauth, saml (optional), httpd, and oxtrust nodes.

Changelog for v0.4.0 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-040).

## Version 0.3.3-12

Steps to migrate to Gluu Cluster v0.3.3-12:

1.  Update `gluu-flask` via `apt-get update && apt-get install -y gluu-flask`.
2.  Remove ldap, oxauth, httpd and oxtrust nodes (containers).
3.  Remove gluuoxauth and gluuoxtrust images via `docker rmi` command.
4.  Re-deploy ldap, oxauth, httpd, and oxtrust nodes.
5.  Install gluu-agent: `apt-get update && apt-get install gluu-agent`.

Changelog for v0.3.3 is available [here](https://github.com/GluuFederation/gluu-flask/blob/master/CHANGES.md#version-033).
