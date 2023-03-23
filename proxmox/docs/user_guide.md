# User guide for **proxmox**

This file is intended for system operators who use the Ansible role [proxmox](https://<git_url>/roles/proxmox).
*The operator is required to have a basic understanding about Ansible and to be able to use Ansible.*

## Variables

Ensure the default variables meet the environment requirements *(source: [defaults/main.yml](../defaults/main.yml))*.
Many of these have been set as a generic standard, in most cases you will not need to change these.

> It is allowed to change these values based on the requirements of the environment this role is used in. Make sure to keep a detailed track record of any deviations from these defaults for documentation purposes including the cause for deviation. In case of permanent variable overrides, make sure to embed them in the inventory as group- or host vars. Please refer to the [internal Ansible guidelines about variables in the inventory](https://<git_url>/academy/ansible_guidelines).

## Playbook requirements

A role can be executed in many ways. Please refer to [the internal Ansible guidelines about playbooks/plays](https://<git_url>/academy/ansible_guidelines/blob/master/Playbooks_and_Plays.md) and/or [the official Ansible documentation](https://docs.ansible.com/ansible/latest/) for more guidance.

| Playbook parameter | Description |
|--------------------|-------------|
| `become` | Ensure elevated rights are used during the play. |
| `gather_facts` | Gathering of facts is required. |

## Available tags

Tags can be used to specifically target certain tasks and task groups. Please refer to [the internal Ansible guidelines about tags](https://<git_url>/academy/ansible_guidelines/blob/master/Tags.md) for more guidance.

## Tags used

> THIS SECTION WILL BE UPDATED AFTER REFACTORING THE ROLE

| Task tag | Description |
|-----|-------------|
| `proxmox_preflight` | Include task file `preflight.yml` |

## Configuration examples

### Configure datacenter.cfg

Below example sets a few configurable options for the `datacenter.cfg` file.

> See [official documentation](https://pve.proxmox.com/wiki/Manual:_datacenter.cfg) for more configurable options.

```yaml
pve_datacenter_cfg:
  console: html5
  keyboard: en-us
  language: en
```

### Configure HA groups

Below example create a group called `ha_node01` for resources assignment to the `ha-node01` host.

> See the [official documentation](https://pve.proxmox.com/wiki/High_Availability#ha_manager_groups) for more information on this topic.

```yaml
pve_cluster_ha_groups:
  - name: ha_node01
    comment: "My HA group"
    nodes: "ha-node01"
    nofailback: 0
    restricted: 0
```

### Live network reload

Reloading network interfaces whilst being live via the Proxmox VE web UI is possible. To achieve this, install the `ifupdown2` package. Note that this package installation will remove the `ifupdown` package. If needed, you can specify this package using the `pve_extra_packages` variable.

```yaml
pve_extra_packages:
  - ifupdown2
```

### User and ACL Management

The `proxmox` role is capable of managing users and groups within Proxmox VE (_both in single server as well as cluster deployments_). Below example will provide the variable layout.

```yaml
pve_groups:
  - name: Admins
    comment: Administrators of this PVE cluster
  - name: api_users
  - name: test_users
pve_users:
  - name: root@pam
    email: postmaster@pve.example
  - name: username@pam
    email: username@pve.example
    firstname: Firstname
    lastname: Lastname
    groups: [ "Admins" ]
  - name: pveapi@pve
    password: "Proxmox789"
    groups:
      - api_users
  - name: testapi@pve
    password: "Test456"
    enable: no
    groups:
      - api_users
      - test_users
  - name: tempuser@pam
    expire: 1609488000
    groups: [ "test_users" ]
    comment: "Temporary user set to expire on Friday, January 1, 2021 8:00:00 AM GMT"
    email: tempuser@pve.example
    firstname: Test
    lastname: User
```

Documentation on the custom modules involved in user and group management can be found on the following locations:

- [proxmox_user](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/proxmox_user.py)
- [proxmox_group](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/proxmox_group.py)

For managing roles and ACLs, a similar method and module is used. However, the main difference is that most of the parameters only accept lists as show in the below example.

```yaml
pve_roles:
  - name: Monitoring
    privileges:
      - "Sys.Modify"
      - "Sys.Audit"
      - "Datastore.Audit"
      - "VM.Monitor"
      - "VM.Audit"
pve_acls:
  - path: /
    roles: [ "Administrator" ]
    groups: [ "Admins" ]
  - path: /pools/testpool
    roles: [ "PVEAdmin" ]
    users:
      - pveapi@pve
    groups:
      - test_users
```

Documentation on the custom modules involved in role and ACL management can be found on the following locations:

- [proxmox_role](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/proxmox_role.py)
- [proxmox_acl](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/proxmox_acl.py)

Refer to `library/proxmox_role.py` and `library/proxmox_acl.py` for custom modules documentation.

### Storage Management

The `proxmox` role is capable of managing storage withing Proxmox VE (_both in single server as well as cluster deployments_). Currently the only supported storage types are `dir`, `rbd`, `nfs`, `cephfs`, `lvm`,`lvmthin`, and `zfspool` as show in below example.

```yaml
pve_storages:
  - name: dir1
    type: dir
    content: [ "images", "iso", "backup" ]
    path: /ploup
    disable: no
    maxfiles: 4
  - name: ceph1
    type: rbd
    content: [ "images", "rootdir" ]
    nodes: [ "pve-node01.local", "pve-node02.local" ]
    username: admin
    pool: rbd
    krbd: yes
    monhost:
      - 10.0.0.1
      - 10.0.0.2
      - 10.0.0.3
  - name: nfs1
    type: nfs
    content: [ "images", "iso" ]
    server: 172.16.0.1
    export: /data
  - name: lvm1
    type: lvm
    content: [ "images", "rootdir" ]
    vgname: vg1
  - name: lvmthin1
    type: lvmthin
    content: [ "images", "rootdir" ]
    vgname: vg2
    thinpool: data
  - name: cephfs1
    type: cephfs
    content: [ "snippets", "vztmpl", "iso" ]
    nodes: [ "pve-node01.local", "pve-node02.local" ]
    monhost:
      - 10.0.0.1
      - 10.0.0.2
      - 10.0.0.3
  - name: zfs1
    type: zfspool
    content: [ "images", "rootdir" ]
    pool: rpool/data
    sparse: true
```

Currently the `zfspool` type can be used only for `images` and `rootdir` contents.  
If you want to store the other content types on a ZFS volume, you need to specify them with type `dir`, path `/<POOL>/<VOLUME>` and add an entry in `pve_zfs_create_volumes`. This example adds a `iso` storage on a ZFS pool:

```yaml
pve_zfs_create_volumes:
  - rpool/iso
pve_storages:
  - name: iso
    type: dir
    path: /rpool/iso
    content: [ "iso" ]
```

Documentation on the custom module involved in storage management can be found on the following location:

- [proxmox_storage](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/proxmox_storage.py)

### Ceph configuration

The `proxmox` role is capable of managing Ceph on your Proxmox hosts, however it still is still **experimental** and should only be used after proper testing.

The following definitions show some of the configurations that are possible:

```yaml
pve_ceph_enabled: true
pve_ceph_network: '172.10.0.0/24'
pve_ceph_cluster_network: '172.10.1.0/24'
pve_ceph_nodes: "ceph_nodes"
pve_ceph_osds:
  # OSD with everything on the same device
  - device: /dev/sdc
  # OSD with block.db/WAL on another device
  - device: /dev/sdd
    block.db: /dev/sdb1
  # encrypted OSD with everything on the same device
  - device: /dev/sdc
    encrypted: true
  # encrypted OSD with block.db/WAL on another device
  - device: /dev/sdd
    block.db: /dev/sdb1
    encrypted: true
# Crush rules for different storage classes
# By default 'type' is set to host, you can find valid types at
# (https://docs.ceph.com/en/latest/rados/operations/crush-map/)
# listed under 'TYPES AND BUCKETS'
pve_ceph_crush_rules:
  - name: replicated_rule
    type: osd # This is an example of how you can override a pre-existing rule
  - name: ssd
    class: ssd
    type: osd
    min-size: 2
    max-size: 8
  - name: hdd
    class: hdd
    type: host
# 2 Ceph pools for VM disks which will also be defined as Proxmox storages
# Using different CRUSH rules
pve_ceph_pools:
  - name: ssd
    pgs: 128
    rule: ssd
    application: rbd
    storage: true
# This Ceph pool uses custom size/replication values
  - name: hdd
    pgs: 32
    rule: hdd
    application: rbd
    storage: true
    size: 2
    min-size: 1
# This Ceph pool uses custom autoscale mode : "off" | "on" | "warn"> (default = "warn")
  - name: vm-storage
    pgs: 128
    rule: replicated_rule
    application: rbd
    autoscale_mode: "on"
    storage: true
pve_ceph_fs:
# A CephFS filesystem not defined as a Proxmox storage
  - name: backup
    pgs: 64
    rule: hdd
    storage: false
    mountpoint: /srv/proxmox/backup
```

> `pve_ceph_network` by default uses the `ipaddr` filter, which requires the `netaddr` library to be installed and usable by your Ansible controller.  
> `pve_ceph_nodes` by default uses `pve_group`, this parameter allows to specify on which nodes install Ceph (_e.g. if you don't want to install Ceph on all your nodes_).  
> `pve_ceph_osds` by default creates unencrypted ceph volumes. To use encrypted volumes the parameter `encrypted` has to be set per drive to `true`.

Documentation on the custom module involved in Ceph management can be found on the following location:

- [ceph_volume](https://<git_url>/collections/-/blob/master/ansible_collections/hcs_company/proxmox/plugins/modules/ceph_volume.py)
