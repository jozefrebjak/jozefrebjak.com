---
title: 'How to deploy a Ceph storage cluster'
resources:
  - name: 'featured-image'
    src: 'featured-image-en.png'
categories: ['documentation']
tags: ['ceph', 'installation']
date: 2021-03-22T09:31:44+01:00
draft: false
---

## What is Ceph

**Ceph** is free open-source clustering software that ties together multiple storage servers, each containing large amounts of hard drives. Essentially, Ceph provides object, block and file storage in a single, horizontally scalable cluster, with no single points of failure. A Ceph storage cluster can be easily scaled over time. It may be configured for high-availability by removing single points of failure. It also has many enterprise features including snapshots, thin provisioning, tiering and self-healing capabilities.

## How Ceph Works

A Ceph Storage Cluster consists of multiple types of daemons:

1. Ceph Monitor
2. Ceph OSD Daemon
3. Ceph Manager
4. Ceph Metadata Server

**Ceph Monitor** (ceph-mon)

A Ceph Monitor maintains a master copy of the cluster map. A cluster of Ceph monitors ensures high availability should a monitor daemon fail. Storage cluster clients retrieve a copy of the cluster map from the Ceph Monitor.

**Ceph OSD Daemon** (ceph-osd)

A Ceph OSD Daemon checks its own state and the state of other OSDs and reports back to monitors.

**Ceph Manager** (ceph-mgr)

A Ceph Manager acts as an endpoint for monitoring, orchestration, and plug-in modules.

**Ceph Metadata Server** (ceph-mds)

A Ceph Metadata Server (MDS) manages file metadata when CephFS is used to provide file services.

Storage cluster clients and each Ceph OSD Daemon use the CRUSH algorithm to efficiently compute information about data location, instead of having to depend on a central lookup table.

{{< admonition type=info open=true >}}
If you want to learn more, please check the official {{< link href="https://docs.ceph.com/en/latest/architecture/" content=documentation >}}.
{{< /admonition >}}

When one or more monitors and two or more object storage are deployed, it is known as a **Ceph Storage Cluster**. The file system, object storage, and block devices read and write data to and from the storage cluster.

## Ceph Cluster Creation
