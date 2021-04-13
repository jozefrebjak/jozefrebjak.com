---
title: 'How to deploy a Ceph storage cluster'
resources:
  - name: 'featured-image'
    src: 'featured-image-en.png'
categories: ['documentation']
tags: ['ceph', 'installation']
date: 2021-04-10T09:31:44+01:00
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

**Ceph OSD** (ceph-osd)

A Ceph OSD Daemon checks its own state and the state of other OSDs and reports back to monitors.

**Ceph Manager** (ceph-mgr)

A Ceph Manager acts as an endpoint for monitoring, orchestration, and plug-in modules.

**Ceph Metadata Server** (ceph-mds)

A Ceph Metadata Server (MDS) manages file metadata when CephFS is used to provide file services.

Storage cluster clients and each Ceph OSD Daemon use the {{< link href="https://docs.ceph.com/en/latest/rados/operations/crush-map/" content=CRUSH >}} algorithm to efficiently compute information about data location, instead of having to depend on a central lookup table.

{{< admonition type=info open=true >}}
If you want to learn more, please check the official {{< link href="https://docs.ceph.com/en/latest/architecture/" content=documentation >}}.
{{< /admonition >}}

When one or more monitors and two or more object storage are deployed, it is known as a **Ceph Storage Cluster**. The file system, object storage, and block devices read and write data to and from the storage cluster.

## Ceph Pacific Cluster Creation

To start and test how it all works, we basically need 3 nodes for the LAB. In our case, we will use virtual servers in a VMware environment.

Each server will have a minimum of virtual resources:

- 2 vCPU
- 4GB RAM
- 2x vDisk (30GB OS disk and 20GB disk as OSD for Ceph)
- Ubuntu 20.04

|   Hostname   |  IP Adress   |       Components        |
| :----------: | :----------: | :---------------------: |
| ceph-node-01 | 10.99.107.81 | Ceph MON, MGR, OSD, MDS |
| ceph-node-02 | 10.99.107.82 | Ceph MON, MGR, OSD, MDS |
| ceph-node-03 | 10.99.107.83 | Ceph MON, MGR, OSD, MDS |

Cephadm creates a new Ceph cluster by “bootstrapping” on a single host, expanding the cluster to encompass any additional hosts, and then deploying the needed services.

- Python 3
- Systemd
- Podman or Docker for running containers
- Time synchronization (such as chrony or NTP)
- LVM2 for provisioning storage devices

Any modern Linux distribution should be sufficient. Dependencies are installed automatically by the bootstrap process.

## Preparation

Since we live in modern times, we have the **ansible** tool at our disposal, which simplifies the whole process and can provide us with everything we need.

Log in to ceph-node-01, which will be our ADMIN node using SSH:

```sh
ssh root@ceph-node-01
```

We update the `/etc/hosts` file with entries for all IP addresses and hostnames.

```sh
10.99.107.81  ceph-node-01
10.99.107.82  ceph-node-02
10.99.107.83  ceph-node-03
```

Update OS:

```sh
apt update && apt -y upgrade
```

Install Ansible and other basic utilities:

```sh
apt update
apt -y install software-properties-common git curl bash-completion ansible
```

Ensure `/usr/local/bin` path is added to PATH.

```sh
echo "PATH=\$PATH:/usr/local/bin" >>~/.bashrc
source ~/.bashrc
```

Generate SSH keys:

```sh
ssh-keygen -t rsa -b 4096 -N '' -f ~/.ssh/id_rsa
```

Upload the SSH key to all nodes using a simple BASH script:

```bash
while read SERVER
do
    ssh-copy-id root@"${SERVER}"
done <<\EOF
ceph-node-01
ceph-node-02
ceph-node-03
EOF
```

Install `cephadm`:

```sh
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
chmod +x cephadm
sudo mv cephadm  /usr/local/bin/
```

Make sure `cephadm` is available for local use:

```sh
cephadm --help
```

With the first node `ceph-node-01` configured, we will create the corresponding Ansible playbook for updating all nodes and upload the SSH public key and the update file `/etc/hosts` to all nodes.

```sh
cd ~/
nano prepare-ceph-nodes.yml
```

We'll edit the content below to have the correct time zone and add it to the file:

```yaml
---
- name: Prepare ceph nodes
  hosts: ceph_nodes
  become: yes
  become_method: sudo
  tasks:
    - name: Set timezone
      timezone:
        name: Europe/Bratislava

    - name: Update system
      apt:
        name: '*'
        state: latest
        update_cache: yes

    - name: Install common packages
      apt:
        name: [nano, git, bash-completion, wget, curl, chrony]
        state: present
        update_cache: yes

    - name: Install Docker
      shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" > /etc/apt/sources.list.d/docker-ce.list
        apt update
        apt install -qq -y docker-ce docker-ce-cli containerd.io
```

Create inventory file.

```sh
nano hosts
```

Insert:

```sh
[ceph_nodes]
ceph-node-01
ceph-node-02
ceph-node-03
```

Configure SSH:

```sh
tee -a ~/.ssh/config<<EOF
Host *
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    IdentitiesOnly yes
    ConnectTimeout 0
    ServerAliveInterval 300
EOF
```

Execute Ansible playbook which will install on all nodes `Chrony` (TZ Europe / Bratislava) and` Docker`:

```sh
ansible-playbook -i hosts prepare-ceph-nodes.yml --private-key ~/.ssh/id_rsa
```

Update `/etc/hosts` on all nodes using Ansible:
`
``sh
nano update-hosts.yml

````

Insert:

```yaml
---
- name: Prepare ceph nodes
  hosts: ceph_nodes
  become: yes
  become_method: sudo
  tasks:
    - name: Clean /etc/hosts file
      copy:
        content: ''
        dest: /etc/hosts

    - name: Update /etc/hosts file
      blockinfile:
        path: /etc/hosts
        block: |
          127.0.0.1     localhost
          10.99.107.81  ceph-node-01
          10.99.107.82  ceph-node-02
          10.99.107.83  ceph-node-03
````

Execute playbook:

```sh
ansible-playbook -i hosts update-hosts.yml --private-key ~/.ssh/id_rsa
```

{{< admonition type=success open=true >}}
At this moment we have the nodes ready for Ceph Cluster! The rest will no longer be so demanding.{{< /admonition >}}

### Install

Create a folder for the CEPH configuration:

```sh
mkdir -p /etc/ceph
```

Prepare the YAML configuration `cluster.yaml` for the CEPH bootstrap:

```yaml
---
service_type: host
addr: ceph-node-01
hostname: ceph-node-01
labels:
  - mon
  - mgr
  - osd
---
service_type: host
addr: ceph-node-02
hostname: ceph-node-02
labels:
  - mon
  - mgr
  - osd
---
service_type: host
addr: ceph-node-03
hostname: ceph-node-03
labels:
  - mon
  - mgr
  - osd
---
service_type: mon
placement:
  hosts:
    - ceph-node-01
    - ceph-node-02
    - ceph-node-03
---
service_type: mgr
placement:
  hosts:
    - ceph-node-01
    - ceph-node-02
    - ceph-node-03
---
service_type: osd
service_id: default_drive_group
placement:
  host_pattern: 'node*'
data_devices:
  all: true
```

The first step in creating a new Ceph cluster is running the cephadm bootstrap command on the Ceph cluster’s first host.

```sh
cephadm bootstrap \
	--mon-ip 10.99.107.81 \
	--initial-dashboard-user admin \
	--initial-dashboard-password Str0ngAdminP@ssw0rd \
    --apply-spec cluster.yml
```

This command will:

- Create a monitor and manager daemon for the new cluster on the all hosts.
- Generate a new SSH key for the Ceph cluster and add it to the root user’s `/root/.ssh/authorized_keys` file.
- Write a minimal configuration file to `/etc/ceph/ceph.conf`. This file is needed to communicate with the new cluster.
- Write a copy of the client.admin administrative (privileged!) secret key to `/etc/ceph/ceph.client`.admin.keyring.
- Write a copy of the public key to `/etc/ceph/ceph.pub`.

Execution output:

```sh
Ceph Dashboard is now available at:

	     URL: https://ceph-node-01:8443/
	    User: admin
	Password: Str0ngAdminP@ssw0rd

You can access the Ceph CLI with:

	sudo /usr/local/bin/cephadm shell --fsid c04bc68a-51b4-11eb-a3a2-77b0d3348620 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Please consider enabling telemetry to help improve Ceph:

	ceph telemetry on

For more information see:

	https://docs.ceph.com/docs/master/mgr/telemetry/
```

If everything is done and Bootstrap has been successful we can install the tools needed for CEPH:

You can install the ceph-common package, which contains all of the ceph commands, including ceph, rbd, mount.ceph (for mounting CephFS file systems), etc.:

```sh
cephadm add-repo --release pacific
cephadm install ceph-common
```

Confirm that the ceph command is accessible with:

```sh
ceph -v
```

Confirm that the ceph command can connect to the cluster and also its status with:

```sh
ceph status
```

## Conclusion

And that’s how we can deploy a fully functional Ceph cluster. There is important flexibility when running Ceph with containers, mainly in terms of upgrades, failures, etc. Cephadm installs and manages a Ceph cluster using containers and systemd, with tight integration with the CLI and dashboard GUI. Cephadm is fully integrated with the new orchestration API and fully supports the new CLI and dashboard features to manage cluster deployment.
