# Install guide for **proxmox**

This file is intended for system engineers who use the Ansible role [proxmox](https://<git_url>/roles/proxmox).
*The engineer is required to have a more thorough understanding about Ansible and to be able to use Ansible.*

## Tasks

All tasks are initiated sequentially from a single source, [tasks/main.yml](../tasks/main.yml). It imports other task files which have been created to each perform a specific stage/phase as part of installing and configuring **proxmox**. All these separate task files combined together make up the entire role resulting in an effective, manageable, modular and idempotent automation sequence to rely on.

| Sequence | Task file | Task tag | Description |
|----------|-----------|----------|-------------|
| 0 | [`main.yml`](../tasks/main.yml) |  | Main task file, called for once the Ansible role is initiated. |
| 1 | [`preflight.yml`](../tasks/preflight.yml) | `proxmox_preflight` | Harden with kernel modules. |

> For a detailed task overview, checkout the task file regarding the specific stage/phase.
> The tasks contain the *yaml* format making it more human readable.

## Deploying a Proxmox VE 7.x cluster

This section will provide and example on how to perform a fully-featured Proxmox VE 7.x cluster from scratch.

To install and configure Proxmox VE, create the appropriate group_vars file (i.e. `group_vars/proxmox`) and modify accordingly for your environment:

```yaml
---
pve_group: proxmox
pve_watchdog: ipmi
pve_ssl_private_key: "{{ lookup('file', pve_group + '/' + inventory_hostname + '.key') }}"
pve_ssl_certificate: "{{ lookup('file', pve_group + '/' + inventory_hostname + '.pem') }}"
pve_cluster_enabled: yes
pve_groups:
  - name: ops
    comment: Operations Team
pve_users:
  - name: admin1@pam
    email: admin1@lab.local
    firstname: Admin
    lastname: User 1
    groups: [ "ops" ]
  - name: admin2@pam
    email: admin2@lab.local
    firstname: Admin
    lastname: User 2
    groups: [ "ops" ]
pve_acls:
  - path: /
    roles: [ "Administrator" ]
    groups: [ "ops" ]
pve_storages:
  - name: localdir
    type: dir
    content: [ "images", "iso", "backup" ]
    path: /plop
    maxfiles: 4
pve_ssh_port: 22

interfaces_template: "interfaces-{{ pve_group }}.j2"

...

```

Variable `pve_group` is set to the group name of our cluster, `proxmox` - it will be used for the purposes of ensuring all hosts within that group can connect to each other and are clustered together. Note that the Proxmox VE cluster name will be set to this group name as well, unless otherwise specified by `pve_cluster_clustername`. Leaving this undefined will default to `proxmox`.

Variable `pve_watchdog` enables IPMI (_hardware_) watchdog support and configures Proxmox VE's HA manager to use it. Leave this undefined if you wish to keep the default NMI (_software_) watchdog.

Variables `pve_ssl_private_key` and `pve_ssl_certificate` point to the SSL certificates for the cluster. In the above example, a file lookup is used to read the contents of an existing file in the playbook, e.g. `files/proxmox/lab-node01.key`. Keep this in mind when using another role like `cert_tool` for certificate management.

Variable `pve_cluster_enabled` enables the role to perform all cluster management tasks. This includes creating a cluster if it doesn't exist, or adding nodes to the existing cluster. There are checks to ensure nodes that are already in existing clusters are mixed with different names.

Variables `pve_groups`, `pve_users`, and `pve_acls` authorizes some local UNIX users (**_they must already exist_**) to access Proxmox VE and gives them the Administrator role as part of the `ops` group. Read the **User and ACL Management** section in the [docs/user_guide.md](user_guide.md) for more info and examples.

Variable `pve_storages` allows to create different types of storage and configure them. The backend needs to be supported by Proxmox; see [official documentation](https://pve.proxmox.com/pve-docs/chapter-pvesm.html). Read the **Storage Management** section in the [docs/user_guide.md](user_guide.md) for more info and examples.

Variable `pve_ssh_port` allows you to change the SSH port. If your SSH is listening on a port other than the default 22, please set this variable. If a new node is joining the cluster, the Proxmox VE cluster needs to communicate once via SSH.

Variable `pve_manage_ssh` (_default ```true```_) allows you to disable any changes this module would make to your SSH server config. This is useful if you use another role to manage your SSH server. Note that setting this to ```false``` is not officially supported, you're on your own to replicate the changes normally made in `ssh_cluster_config.yml`.

Variable `interfaces_template` is set to the path of a template we'll use for configuring the network on these Debian machines. This is only necessary if you want to manage networking from Ansible rather than manually or via each host in Proxmox VE. You must be familiar with Ansible prior to doing this, as your method may involve setting host variables for the IP addresses for each host, etc.

Let's get that interface template out of the way. Feel free to skip this file (and leave it undefined in `group_vars/proxmox`) otherwise. Here's an example:

```shell
# {{ ansible_managed }}
auto lo
iface lo inet loopback

allow-hotplug enp2s0f0
iface enp2s0f0 inet manual

auto vmbr0
iface vmbr0 inet static
    address {{ lookup('dig', ansible_fqdn) }}
    gateway 10.4.0.1
    netmask 255.255.255.0
    bridge_ports enp2s0f0
    bridge_stp off
    bridge_fd 0

allow-hotplug enp2s0f1
auto enp2s0f1
iface enp2s0f1 inet static
    address {{ lookup('dig', ansible_hostname + "-clusternet.local") }}
    netmask 255.255.255.0
```

The above example uses the `dig` command to do a DNS record lookup for each machine. Asserted for this example is that a DNS server is available and reachable in the environment. After the lookup it will be configured as a bridge for the VM instances. The above is just a mere example, feel free to create your own. Especially when using bonding, multiple networks for management/corosync, storage and VM traffic, etc.

Lastly, create the installation playbook e.g. `setup_proxmox.yml`:

```yaml
---
- hosts: proxmox
  become: true
  roles:
    - network_base
    - network_time

# Leave this out if you're not modifying networking through Ansible
- hosts: proxmox
  become: true
  serial: 1
  tasks:
    - name: Install bridge-utils
      apt:
        name: bridge-utils

    - name: Configure /etc/network/interfaces
      template:
        src: "{{ interfaces_template }}"
        dest: /etc/network/interfaces
      register: _configure_interfaces

    - block:
      - name: Reboot for networking changes
        shell: "sleep 5 && shutdown -r now 'Networking changes found, rebooting'"
        async: 1
        poll: 0

      - name: Wait for server to come back online
        wait_for_connection:
          delay: 15
      when: _configure_interfaces is changed

- hosts: proxmox
  become: true
  roles:
    - proxmox

...

```

Basically, we run the `network_base` and `network_time` roles across all hosts, configure networking on `proxmox` with our separate cluster network and bridge layout, reboot to make those changes take effect, and then run this Proxmox role against the hosts to setup a cluster.

At this point, the example playbook is ready to be ran with Ansible. Before actually running the playbook, we need to ensure that dependencies are installed:

```shell
ansible-galaxy install -r roles/requirements.yml --force
dnf install python3-jmespath -y
```

Python library `jmespath` is required for some of the tasks involving clustering. Next, we pass `--force` to `ansible-galaxy install` to ensure all required roles are updated to their latest versions if already installed.

Next, time to actually run the example playbook and configure our first Proxmox VE cluster:

```shell
ansible-playbook -i inventory setup_proxmox.yml -e '{"pve_reboot_on_kernel_update": true}'
```

The extra variable given with `-e '{"pve_reboot_on_kernel_update": true}'` should mainly be run the first time you do the Proxmox cluster setup, as it'll reboot the server to boot into a Proxmox VE kernel. Subsequent runs should leave this out, as you want to sequentially reboot servers after the cluster is running.

> Note that when Proxmox VE is installed using the suppliers ISO, you're already on the Proxmox VE kernel and rebooting is not needed for the initial setup.

To specify a particular user, use `-u root` (replacing `root`), and if you need to provide passwords, use `-k` for SSH password and/or `-K` for sudo password; in example:

```shell
ansible-playbook -i inventory setup_proxmox.yml -K -u admin1
```

This will ask for a sudo password, then login to the `admin1` user (using public key auth - add `-k` for password) and run the playbook.

After the initial setup, Ceph storage can be deployed as well as other management and configurational tasks.
