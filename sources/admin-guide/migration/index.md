# Migration

[TOC]

## Migrating to v0.3.3-12

What's new in v0.3.3-12:

*   oxAuth v2.3.4
*   oxTrust v2.3.4

Removed in v0.3.3-12:

*   Built-in recovery command is no longer available and replaced by gluu-agent.

Steps to migrate to Gluu Cluster v0.3.3-12:

1.  Remove oxauth and oxtrust nodes (containers).
2.  Remove gluuoxauth and gluuoxtrust images.
3.  Re-deploy oxauth and oxtrust nodes.
4.  Install gluu-agent: `apt-get update && apt-get install gluu-agent`.
