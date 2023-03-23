# Proxmox VE

## Description

Installs and configures Proxmox Virtual Environment 7.x on Debian servers.

This role allows you to deploy and manage single-node PVE installations and PVE clusters (3+ nodes) on Debian 11 - Bullseye.  
The following can be configured by using this role:

- PVE RBAC definitions (_roles, groups, users, and access control lists_)
- PVE Storage definitions
- PVE Datacenter configuration
- HTTPS certificates for the Proxmox Web GUI (BYO)
- PVE repository selection (e.g. `pve-no-subscription` or `pve-enterprise`)
- Watchdog modules (_IPMI and NMI_) with applicable `pve-ha-manager` config
- ZFS module setup and ZED notification email

With clustering enabled, this role allows you to do the following:

- Ensure all hosts can connect to one another as root over SSH
- Initialize a new PVE cluster (_or possibly adopt an existing one_)
- Create or add new nodes to a PVE cluster
- Setup Ceph on a PVE cluster
- Create and manage high availability groups

## Guidelines

This Ansible role is part of the basic Ansible guidelines and style methods defined here:
[Ansible Guidelines](<url_to_be_provided>)

All changes on servers should trigger an alarm except for the configured exceptions. This can be done with Aide.

## Instructions and Guides

| Guide | Description |
|-------|-------------|
| [User guide](docs/user_guide.md) | Guide for system operators who use the Ansible role |
| [Install guide](docs/install_guide.md) | Guide for system engineers who use the Ansible role |
| [Admin guide](docs/admin_guide.md) | Guide for system engineers who modify the Ansible role |

## Example Playbook

This will configure hosts in the group `pve01` as one cluster, as well as
reboot the machines should the kernel have been updated. (Only recommended to
set this flag during installation - reboots during operation should occur
serially during a maintenance period.) It will also enable the IPMI watchdog.

```yaml
---
- hosts: pve01
  become: true
  roles:
    - role: proxmox
        pve_group: pve01
        pve_cluster_enabled: yes
        pve_reboot_on_kernel_update: true
        pve_watchdog: ipmi

...

```

## Requirements & dependencies

| Sequence | Ansible role | Comment |
|----------|--------------|---------|
| 1 | `hosts_file` | The hosts file acts as a DNS fallback in order to resolve hostnames. |
| 2 | `yum_repos` or `satellite_client` | Package repositories are required to install/update software. |
| 3 | `aide` | Integrity monitoring for **aide** will be initiated after this role has run.  Run playbook with `aide_update=false` to skip this task |
| 4 | `network_time` | Network Time Protocol is required by Proxmox VE. |

When clustering is enabled, this role makes use of the `json_query` filter, which requires that the `jmespath` library be installed on your control host.  
To achieve this you can either `pip install jmespath` or install it via your distribution's package manager, e.g. `dnf install python3-jmespath -y`.

## Changelog

For the actual version and change history please read the [changelog](CHANGELOG.md).
