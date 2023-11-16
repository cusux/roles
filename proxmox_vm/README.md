# Proxmox VM

## Description

Installs and configures virtual machines (_KVM_) on a Proxmox Virtual Environment 7.x.

This role allows you to deploy and manage virual machines (_kvm_) on both single-node as well as clustered Proxmox VE 7.x instances.  
The following can be configured by using this role:

- Virtual machine templates
- Virtual machines from a template
- VM network stacks including interface (re)naming
- VM storage, memory, cores and cpu
- VM users and ssh keys
- Custom scripts to run on VM after first boot
- Automatic updates after VM is deployed

## Guidelines

This Ansible role is part of the basic Ansible guidelines and style methods defined here:
[Ansible Guidelines](https://xffgit1t.xfftest.lan/kpn/academy/ansible_guidelines)

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
- name: Playbook to Proxmox VE virtual machines
  hosts:
    - proxmox
  become: true
  gather_facts: true

  roles:
    - proxmox_vm

...

```

## Requirements & dependencies

This role makes use of the `json_query` filter, which requires that the `jmespath` library be installed on your control host.  
To achieve this you can either `pip install jmespath` or install it via your distribution's package manager, e.g. `dnf install python3-jmespath -y`.

## Changelog

For the actual version and change history please read the [changelog](CHANGELOG.md).
