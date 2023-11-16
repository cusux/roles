# User guide for **proxmox_vm**

This file is intended for system operators who use the Ansible role [proxmox_vm](<url_to_be_provided>/roles/proxmox_vm).
*The operator is required to have a basic understanding about Ansible and to be able to use Ansible.*

## Variables

Ensure the default variables meet the environment requirements *(source: [defaults/main.yml](../defaults/main.yml))*, or overwrite them using the proper `group_vars`.
Some of these have been set as a generic standard, in those most cases you most likely will not need to change these.

> It is allowed to change these values based on the requirements of the environment this role is used in. Make sure to keep a detailed track record of any deviations from these defaults for documentation purposes including the cause for deviation. In case of permanent variable overrides, make sure to embed them in the inventory as group- or host vars. Please refer to the [internal Ansible guidelines about variables in the inventory](<url_to_be_provided>/academy/ansible_guidelines).

## Playbook requirements

A role can be executed in many ways. Please refer to [the internal Ansible guidelines about playbooks/plays](<url_to_be_provided>/academy/ansible_guidelines/blob/master/Playbooks_and_Plays.md) and/or [the official Ansible documentation](https://docs.ansible.com/ansible/latest/) for more guidance.

| Playbook parameter | Description |
|--------------------|-------------|
| `become` | Ensure elevated rights are used during the play. |
| `gather_facts` | Gathering of facts is required. |

## Available tags

Tags can be used to specifically target certain tasks and task groups. Please refer to [the internal Ansible guidelines about tags](<url_to_be_provided>/academy/ansible_guidelines/blob/master/Tags.md) for more guidance.

## Tags used

| Task tag | Description |
|-----|-------------|
| `proxmox_preflight` | Include task file `preflight.yml` |
| `proxmox_vm_template` | Inlcude task file `templates.yml` |
| `proxmox_vm_clone` | Include task file `clone.yml` |
| `proxmox_vm_configure` | Include task file `configure.yml` |
| `proxmox_vm_manage_states` | Include task file `manage_vm_state.yml` |

## Configuration examples

### Configure group and host variables

Configuration of `group_vars` and `host_vars` is, due to naming convention, self-declarative. However, to have a proper understanding of the variables and configurations refer to the [install guide](install_guide.md) for an explanatory on the configurable options.
