# Administration guide for **proxmox**

This file is intended for system engineers who modify the Ansible role [proxmox](<url_to_be_provided>/roles/proxmox).
*The engineer is required to have an advanced understanding about Ansible and to be able to create/modify Ansible roles.*

## Git

The entire Ansible implementation and life-cycle is subject to the [Git Workflow](<url_to_be_provided>). All modifications have to follow the workflow in order to be more manageable, measurable and maintainable including a track record of changes. In addition to the workflow, [release management guidelines](<url_to_be_provided>) are in affect for every type of release regarding the evolutionary cycle of the Ansible implementation.

## Editable Proxmox sections

A lot of stuff is going on when installing or configuring a Proxmox environment. This is why it is split up in sections.
Each section that is allowed for custom adjustments is explained here below synchronously to the sections in the [defaults/main.yml](../defaults/main.yml) file. In the [docs/user_guide.md](user_guide.md) examples will be provided on how to implement the variables.

### OS Repository URL

Variable `pve_os_repository` is set to the URL of the OS repository in the network.

```yaml
pve_os_repository: https://os_repo_url
```

### PVE group

Variable `pve_group` is set to the group name of our cluster, `proxmox` - it will be used for the purposes of ensuring all hosts within that group can connect to each other and are clustered together. Note that the PVE cluster name will be set to this group name as well, unless otherwise specified by `pve_cluster_clustername`.

```yaml
pve_group: "<string>"
```

By default this variable is set to ```proxmox```.

### PVE repository

Variable `pve_repository_line` is set to the enterprise or community repository location for the Proxmox instance.

```yaml
pve_repository_line: "<string>"
```

By default this variable is set to ```deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription```.  

### No subscription warning

Variable `pve_remove_subscription_warning` defines wether or not to patch the subscription warning messages in Proxmox when using the community edition.

```yaml
pve_remove_subscription_warning: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### Extra packages

Variable `pve_extra_packages` defines which extra packages to install e.g. `ifupdown2`.

```yaml
pve_extra_packages:
  - ifupdown2
```

By default this variable is set to an empty list object ```[]```.

### Run system upgrades

Variable `pve_run_system_upgrades` defines wether or not to let the role perform system upgrades.

```yaml
pve_run_system_upgrades: [ <true> | <false> ]
```

By default this variable is set to ```false```.

### SSH port configuration

Variable `pve_ssh_port` defines the listening port for the SSH daemons. If a new node is joining the cluster, the PVE cluster needs to communicate once over SSH.

```yaml
pve_ssh_port: <integer>
```

By default this variable is set to ```22```.

### SSH configuration management

Variable `pve_manage_ssh` defines wether or not this role will manage the SSH configuration. Set to `false` if you have another role managing the SSH configuration.

> When you are using another role to manage the SSH configuration, you must manually replicate all changes to the `ssh_cluster_config.yml` file!

```yaml
pve_manage_ssh: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### Cluster configuration

Variables `pve_cluster_enabled`, `pve_cluster_clustername` and `pve_manage_hosts_enabled` are used to define clustering options:

- define wether or not to enable clustering
- define the clustername of the PVE cluster
- define wether or not to configure the hosts file

```yaml
pve_cluster_enabled: [ <true> | <false> ]
pve_cluster_clustername: "<string>"
pve_manage_hosts_enabled : [ <true> | <false> ]
```

By default these variables are set to:

```yaml
pve_cluster_enabled: false
pve_cluster_clustername: "{{ pve_group }}"
pve_manage_hosts_enabled: true
```

### Cluster corosync network configuration

Variables `pve_cluster_addr0` and `pve_cluster_addr1` are used to provide networking information to Corosync.

> depending on the Proxmox VE version, configure `ring0_addr/ring1_addr` or `link0_addr/link1_addr`. For more information see [official documentation](https://pve.proxmox.com/pve-docs/chapter-pvecm.html#_separate_cluster_network).

```yaml
pve_cluster_addr0: "<ip address>"
pve_cluster_addr1: "<ip address>"
```

By default these variables are set to:

```yaml
pve_cluster_addr0: "{{ ansible_default_ipv4.address }}"
# pve_cluster_addr1: "another interface's IP address or hostname"
```

### HA configuration

Variable `pve_cluster_ha_groups` defines the list of HA groups to create in Proxmox VE.

> See [official documentation](https://pve.proxmox.com/wiki/High_Availability#ha_manager_groups) for more information.

```yaml
pve_cluster_ha_groups: "<list of ha groups>"
```

By default this variable is set to ```[]```.

### Proxmox upgrades

Variable `pve_run_proxmox_upgrades` defines wether or not this role will manage the Proxmox VE upgrades.

```yaml
pve_run_proxmox_upgrades: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### Kernel updates

Variables `pve_check_for_kernel_update`, `pve_reboot_on_kernel_update`, `pve_reboot_on_kernel_update_delay` and `pve_remove_old_kernels` define wether or not to:

- check for kernel updates
- force a reboot after kernel update
- use a delay in seconds before rebooting after a kernel update
- remove old kernel versions

```yaml
pve_check_for_kernel_update: [ <true> | <false> ]
pve_reboot_on_kernel_update: [ <true> | <false> ]
pve_reboot_on_kernel_update_delay: <integer>
pve_remove_old_kernels: [ <true> | <false> ]
```

By default these variables are set to:

```yaml
pve_check_for_kernel_update: true
pve_reboot_on_kernel_update: false
pve_reboot_on_kernel_update_delay: 60
pve_remove_old_kernels: true
```

### Hardware watchdog

Proxmox VE uses a software watchdog (`nmi_watchdog`) by default. Variables `pve_watchdog`, `pve_watchdog_ipmi_action` and `pve_watchdog_ipmi_timeout` are used to configure the watchdog:

- switch between software and hardware watchdog
- watchdog action
- watchdog action timeout in seconds

```yaml
pve_watchdog: [ <none> | <ipmi> ]
pve_watchdog_ipmi_action: [ <reset> | <power_cycle> | <power_off> ]
pve_watchdog_ipmi_timeout: <integer>
```

By default these variables are set to:

```yaml
pve_watchdog: none
pve_watchdog_ipmi_action: power_cycle
pve_watchdog_ipmi_timeout: 10
```

### ZFS configuration

Variables `pve_zfs_enabled`, `pve_zfs_options`, `pve_zfs_zed_email`, `pve_zfs_create_volumes` and `pve_hooks` are used to configure ZFS options:

- wether or not to use ZFS and thus install and configure ZFS packages
- modprobe parameters to pass to ZFS module on boot/modprobe; see the [official documentation](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Module%20Parameters.html)
- e-mail address to send ZFS notifications to
- list of ZFS volumes to create as PVE Storages
- dictionary of ZFS post install hook tasks

```yaml
pve_zfs_enabled: [ <true> | <false> ]
pve_zfs_options: "<see_documentation>"
pve_zfs_zed_email: "<user@host.domain>"
pve_zfs_create_volumes: [ "<list of volumes>" ]
pve_hooks: { "<zfs_post_install>" }
```

By default these variables are set to:

```yaml
pve_zfs_enabled: false
pve_zfs_options: ""
pve_zfs_zed_email: ""
pve_zfs_create_volumes: []
pve_hooks: {}
```

### Ceph basic configuraton

Variables `pve_ceph_enabled`, `pve_ceph_repository_line`, `pve_ceph_network` and `pve_ceph_cluster_network` are used for the basic Ceph configurations:

- specify wether or not to install and configurare Ceph packages
- specify the Ceph apt repository; see the [official documentation](https://pve.proxmox.com/wiki/Package_Repositories)
- specify the Ceph public network
- specify the Ceph cluster network when it differs from the public network; see the [official documentation](https://pve.proxmox.com/pve-docs/chapter-pveceph.html#pve_ceph_install_wizard)

```yaml
pve_ceph_enabled: [ "<true" | "<false>" ]
pve_ceph_repository_line: "<string>"
pve_ceph_network: "<subnet>"
pve_ceph_cluster_network: "<subnet>"
```

By default these variables are set to:

```yaml
pve_ceph_enabled: false
pve_ceph_repository_line: "deb http://download.proxmox.com/debian/ceph-pacific bullseye main"
pve_ceph_network: "{{ (ansible_default_ipv4.network +'/'+ ansible_default_ipv4.netmask) | ipaddr('net') }}"
pve_ceph_cluster_network: ""
```

### Ceph node configuratons

Variables `pve_ceph_nodes`, `pve_ceph_mon_group`, `pve_ceph_mgr_group` and `pve_ceph_mds_group` are used for the Ceph nodes configurations:

- define the Ansible hostgroup containing all Ceph nodes
- define the Ansible hostgroup containing all Ceph monitor nodes
- define the Ansible hostgroup containing all Ceph manager nodes
- define the Ansible hostgroup containing all Ceph metadate server hosts

```yaml
pve_ceph_nodes: "<string>"
pve_ceph_mon_group: "<string>"
pve_ceph_mgr_group: "<string>"
pve_ceph_mds_group: "<string>"
```

By default these variables are set to:

```yaml
pve_ceph_nodes: "{{ pve_group }}"
pve_ceph_mon_group: "{{ pve_group }}"
pve_ceph_mgr_group: "{{ pve_ceph_mon_group }}"
pve_ceph_mds_group: "{{ pve_group }}"
```

### Ceph storage configurations

Variables `pve_ceph_osds`, `pve_ceph_pools`, `pve_ceph_fs` and `pve_ceph_crush_rules` are used to define the Ceph storage configurations:

- list of OSD disks
- list of pools to create
- list of CephFS filesystems to create
- List of CRUSH rules to create

```yaml
pve_ceph_osds: "<list of disks>"
pve_ceph_pools: "<list of pools>"
pve_ceph_fs: "<list of filesystems>"
pve_ceph_crush_rules: "<list of CRUSH rules>"
```

By default these variables are set to ```[]```.

### SSL configuration

Variables `pve_ssl_private_key` and `pve_ssl_certificate` are used to configure HTTPS connections for Proxmox.

> These variables needs to work on conjunction with Ansible role [cert_tool](<url_to_be_provided>/roles/cert_tool). Fetching the proper contents could be done by utilizing the Ansible Jinja2 lookup filter e.g. `{{ lookup('file', pve_group + '/' + inventory_hostname + '.key') }}`.

```yaml
pve_ssl_private_key: "<private key contents>"
pve_ssl_certificate: "<certificate contents>"
```

By default these variables are set to ```""```.

### ACL configurations

Variables `pve_roles`, `pve_groups`, `pve_users` and `pve_acls` are used to configure ACL in Proxmox:

- list of roles with specific privileges
- list of group definitions
- list of user definitions

```yaml
pve_roles: "<list of roles>"
pve_groups: "<list of groups>"
pve_users: "<list of users>"
pve_acls: "<list of acls>"
```

By default these variables are set to ```[]```.

### Other configurations

Variables `pve_storages` and `pve_datacenter_cfg` are used to configure storages and the PVE `datacenter.cfg` file.

> The storages backend need to be supported by Proxmox; see [official documentation](https://pve.proxmox.com/pve-docs/chapter-pvesm.html) for more information.

```yaml
pve_storages: "<list of storages>"
pve_datacenter_cfg: "<dictionary for datacenter.cfg>"
```

By default these variables are set to:

```yaml
pve_storages: []
pve_datacenter_cfg: {}
```

Variables `pve_disable_services` and `pve_disabled_services` are used to define which Proxmox VE services should be disabled. A usecase for this can be a hardening role which sets a custom login banner and thus providing the need to disable the `pvebanner` service daemon.

```yaml
pve_disable_services: [ <true> | <false> ]
pve_disabled_services: "<list of services>"
```

By default these variables are set to:

```yaml
pve_disable_services: false
pve_disabled_services: []
  - name: "pvebanner"
    state: "stopped"
    enabled: "no"
    masked: "yes"
```

A working example of disabling the aforementioned `pvebanner` service daemon is as follows:

```yaml
pve_disable_services: true
pve_disabled_services:
  - name: "pvebanner"
    state: "stopped"
    enabled: "no"
    masked: "yes"
```
