# Install guide for **proxmox_vm**

This file is intended for system engineers who use the Ansible role [proxmox_vm](<url_to_be_provided>/roles/proxmox_vm).
*The engineer is required to have a more thorough understanding about Ansible and to be able to use Ansible.*

## Tasks

All tasks are initiated sequentially from a single source, [tasks/main.yml](../tasks/main.yml). It imports other task files which have been created to each perform a specific stage/phase as part of installing and configuring virtual machines on a Proxmox VE 7.x cluster. All these separate task files combined together make up the entire role resulting in an effective, manageable, modular and idempotent automation sequence to rely on.

| Sequence | Task file | Task tag | Description | Necessity |
|----------|-----------|----------|-------------|----------|
| 0 | [`main.yml`](../tasks/main.yml) |  | Main task file, called for once the Ansible role is initiated. | `mandatory` |
| 1 | [`preflight.yml`](../tasks/preflight.yml) | `always`,`proxmox_vm_preflight` | Task file for preflight checks and balances. | `mandatory` |
| 2 | [`templates.yml`](../tasks/templates.yml) | `proxmox_vm_template` | Task file to download cloud-init image and VM template creation. | `optional` |
| 3 | [`clone.yml`](../tasks/clone.yml) | `proxmox_vm_clone` | Task file to initiate new instances by means of the created template (_cloning_). | `optional` |
| 4 | [`configure.yml`](../tasks/configure.yml) | `proxmox_vm_configure` | Task file to configure the cloned instances. | `optional` |
| 4.1 | [`configure_networks.yml`](../tasks/configure_networks.yml) | | Subtask file to configure networking for the cloned instances. | |
| 4.1.1 | [`configure_ci_templates.yml`](../tasks/configure_ci_templates.yml) | | Subtask file to configure cloud-init templates for the cloned instances. | |
|4.2 | [`configure_vms.yml`](../tasks/configure_vms.yml) | | Subtask file to configure the cloned instances based on the cloud-init templates. | | |
| 5 |[`manage_vm_state.yml`](../tasks/manage_vm_state.yml) | `proxmox_vm_manage_states` | Task file to manage the states of the VM instances. | `optional` |
| 5.1 |[`manage_vm_state_purge.yml`](../tasks/manage_vm_state_purge.yml) | | Sub task file to purge cloud-init templates of removes VM instances. | |

> For a detailed task overview, checkout the task file regarding the specific stage/phase.
> The tasks contain the *yaml* format making it more human readable.

## Creating virtual machines on a Proxmox VE 7.x cluster

This section will provide an example on how to create, configure and manage virtual machines on a Proxmox VE 7.x cluster.

To install, configure and manage Proxmox virtual machines, create the appropriate group_vars file (i.e. `group_vars/proxmox_vm/main.yml`) and modify accordingly for your environment:

```yaml
---
proxmox_vm_remote_repository: "https://myremoterepo.example.com/pub/"
proxmox_vm_pypi_proxmoxer: "proxmoxer-1.2.0.tar.gz"

proxmox_vm_templates_management: true
proxmox_vm_template_remote_image: "rhel-8.5-x86_64-kvm.qcow2"
proxmox_vm_template_remote_image_url: "{{ proxmox_vm_remote_repository }}/{{ proxmox_vm_template_remote_image }}"
proxmox_vm_template_remote_image_download: true
proxmox_vm_template_remote_image_overwrite: false
proxmox_vm_template_remove: false
proxmox_vm_template_name: "rhel85"
proxmox_vm_template_vm_id: '10000'
proxmox_vm_template_memory: '2048'

proxmox_vm_hypervisor: "{{ groups['proxmox'][0] }}"
proxmox_vm_hypervisor_api: "{{ hostvars[groups['proxmox'][0]].ansible_default_ipv4.address }}:443"
proxmox_vm_hypervisor_user: "root"
proxmox_vm_hypervisor_user_realm: "pam"
proxmox_vm_hypervisor_user_password: "mysupersecretpassword"

proxmox_vm_guest_nodes: "{{ groups['proxmox_vms'] }}"

proxmox_vm_ci_username: "ciuser"
proxmox_vm_ci_password: "$6$6wShN0YV2URlEyez$Vf96zbLPqQ.6sDyY7tL5S2Avq.Tn7rRKGDzk8N9CJHkWxsW0tnSvMQCzwZsa2wxfNGYZlklhZtywx5HXsx21p0"
proxmox_vm_ci_ssh_keys: []
proxmox_vm_ci_enable_ssh_password_auth: true
proxmox_vm_enable_qemu_agent: true
proxmox_vm_storage: "local-zfs"
proxmox_vm_resize_disk: true

proxmox_vm_enable_update_after_boot: false

proxmox_vm_run_cmd: []
proxmox_vm_install_packages: []
proxmox_vm_dns_fqdn: "example.com"
proxmox_vm_dns:
  - "10.0.0.254"

proxmox_vm_local_storage_path: "/var/lib/vz"
proxmox_vm_cloudinit_image_location: "{{ proxmox_vm_local_storage_path }}/template/iso"
proxmox_vm_cloudinit_snippets_location: "{{ proxmox_vm_local_storage_path }}/snippets"
proxmox_vm_cloudinit_volume: "local"

proxmox_vm_states_managed: true
proxmox_vm_states_desired: "absent"
proxmox_vm_states_targets: []

...

```

Variable `proxmox_vm_remote_repository` is set to the remote repository from which cloud-init images and, if applicable, proxmoxer are downloaded. This location should be an existing and reachable location. especially when `proxmox_vm_templates_management` and `proxmox_vm_template_remote_image_download` are set to ```true```.

Variable `proxmox_vm_pypi_proxmoxer` is set to the filename of the `proxmoxer` installation packages as presented on the remote repository. This variable is only needed when `proxmoxer` needs to be installed on the primary Proxmox VE hypervisor.

Variable `proxmox_vm_templates_management` defines wether or not to manage the virtual machine template. As stated earlier, this variable goes hand-in-hand with `proxmox_vm_remote_repository`.

Variables `proxmox_vm_template_remote_image` and `proxmox_vm_template_remote_image_download` are used to define the filename of the cloud-init template on the specified remote repository and wether or not to download the remote image. Note that these variables also go hand-in-hand with `proxmox_vm_remote_repository`.

Variable `proxmox_vm_template_remote_image_overwrite` specifies wether or not to overwrite any pre-existing cloud-init images when the filename matches. If left to ```false```, if the remote and local filenames are matching, no download will take place.

Variable `proxmox_vm_template_remove` is used to specify deletion of a pre-existing template with the same name. When set to ```false```, the template will not be removed but updated where applicable.

Variables `proxmox_vm_template_name`, `proxmox_vm_template_vm_id` and `proxmox_vm_template_memory` are used to provide some basic specifications to the virtual machine template. It is adviced to set a declaritive name like ```rhel85```; use a high ID for the vm to ensure it is far enough away from any future virtual machine ids; specify a proper, minimal amount of memory for the virtual machine template. The latter can be overwritten by `hosts_vars` of each virtual machine during configuration.

Variable `proxmox_vm_hypervisor` is the specification of the primary Proxmox node of the cluster. As per example, this variable could be configured as ```{{ groups['proxmox'][0] }}"```.

Variable `proxmox_vm_hypervisor_api` is the API url specification. Depending on the configuration, this value can also be dynamically set using the inventory variables: ```{{ hostvars[groups['proxmox'][0]].ansible_default_ipv4.address }}:443```. Note the port number; this is also depending on the configuration. By default it is ```8006```, in case of an advisable reverse proxy on the hypervisor, this can be any other value.

Variables `proxmox_vm_hypervisor_user`, `proxmox_vm_hypervisor_user_realm` and `proxmox_vm_hypervisor_user_password` are used to provide the user specification of the user which can connect to the Proxmox VE hypervisor. Note that the realm can be set to either a remote authentication provider or the local ```pam``` realm. It is advisable to place the `proxmox_vm_hypervisor_user` and `proxmox_vm_hypervisor_user_password` in a seperate vault file or service.

Variable `proxmox_vm_guest_nodes` is a list object containing hostnames of the virtual machine instances wich need to be managed with this role. Note that for each hostname a valid `host_vars` entry must exist. An example of dynamically setting this parameter is ```{{ groups['proxmox_vms'] }}```.

Variables `proxmox_vm_ci_username`, `proxmox_vm_ci_password` and `proxmox_vm_ci_ssh_keys` are used to feed cloud-init user authentication information into the virtual machines. Note that the `proxmox_vm_ci_password` should be a SHA512 encrypted string. It is advisable to place all three variables in a separate vault file or service.

Variable `proxmox_vm_ci_enable_ssh_password_auth` is used to define wether or not the cloud-init user is able to login on the virtual machine over SSH by means of password authentication. When this parameter is set to ```false```, ensure SSH-keys are provided as described above to prevent lockout.

Variables `proxmox_vm_enable_qemu_agent`, `proxmox_vm_storage` and `proxmox_vm_resize_disk` are user to specify wether or not to use the qemu agent on the virtual machine, define the virtual machines location on the hypervisor and wether or not to resize the virtual machines disk as specified in the `host_vars` of that virtual machine.

Variables `proxmox_vm_enable_update_after_boot`, `proxmox_vm_run_cmd` and `proxmox_vm_install_packages` are used to specify wether or not to update the virtual machine, run specified commands and install specified packages on initial boot of the virtual machine. When `proxmox_vm_states_managed` is set to ```true``` with `proxmox_vm_states_desired` set to ```started```, these actions will be performed directly after virtual machine creation and provides a seamless workflow to get the cluster up and running in it's desired initial state.

Variables `proxmox_vm_dns_fqdn` and `proxmox_vm_dns` define the DNS domain in which the virtual machines reside in and also provide the IP address of the DNS servers to the virtual machines.

Variables `proxmox_vm_local_storage_path`, `proxmox_vm_cloudinit_image_location`, `proxmox_vm_cloudinit_snippets_location` and `proxmox_vm_cloudinit_volume` are pretty much self-declarative; they define the default storage location on the hypervisors in the cluster and specify the paths to ISO files, cloud-init template files (_snippets_) as well as define the used storage volume for cloud-init virtual machines.

Variables `proxmox_vm_states_managed`, `proxmox_vm_states_desired` and `proxmox_vm_states_targets` define wether or not to manage the state of the virtual machines and to which state they must be managed. The desired states can be of values ```started```, ```stopped```, ```restarted``` and ```absent```. The list object `proxmox_vm_states_targets` is used as exception to `proxmox_vm_guest_nodes`, this way a subset of hostnames can be defined to which the desired states must be managed.

> Note that if states are managed and set to ```absent``` without an exception list, virtual machines will be created, configured and destroyed in a single run! So don't come to me complaining when this happens, you should've RTFM.

After setting the `group_vars` for this role, ensure each managable VM is provided the following dictionary object in the `host_vars` of that specific host (i.e. `host_vars/virtualnode01.yml`) and modify it according to your needs:

```yaml
proxmox_vm:
  memory_size: 4096
  balloon: 2048
  cores: 2
  vcpus: 2
  storage: "local-zfs"
  disk_size: "100G"
  name: "virtualnode01"
  proxmox_network_config:
    - name: "net0"
      interface_name: "mgmt0"
      vlan: 1001
      ipv6: true
      ip6: "2001:db8:85a3:8d3:1319::254"
      ip: "172.16.1.10"
      gateway: "172.16.1.254"
      gateway6: "2001:db8:85a3:8d3:1319::1"
      subnet: "/24"
      subnet6: "/64"
      bridge: "vmbr0"
    - name: "net1"
      interface_name: "data0"
      ipv6: false
      ip: "192.168.1.10"
      subnet: "/24"
      bridge: "vmbr1"

...

```

The above dictionary is pretty much self-declarative. Small note on `memory_size` and `balloon` sizes; the memory size is the absolute memory size the VM can claim on the hypervisor. The balloon size is the specification to which memory size the VM can be reduced by giving up unused memory back to the host. This freed memory then can be temporarily assigned to another busy VM. The VM itself decides which processes or cache pages to swap out to free up memory for the balloon. To disable this ballooning feature and driver, set this value to ```0``` for that specific VM.

When each VM is taken into the Ansible inventory, the `group_vars` and `host_vars` files are populated, it is time to create the management playbook e.g. `manage_proxmox_vms.yml`:

```yaml
---
- name: Playbook to manage Proxmox VE virtual machines
  hosts:
    - proxmox_vms
  become: true
  gather_facts: true

  roles:
    - proxmox_vm

...

```

At this point, the example playbook is ready to be ran with Ansible. Before actually running the playbook, we need to ensure that dependencies are installed:

```shell
ansible-galaxy install -r roles/requirements.yml --force
dnf install python3-jmespath -y
```

Python library `jmespath` is required for some of the tasks involving virtual machine states. Next, we pass `--force` to `ansible-galaxy install` to ensure all required roles are updated to their latest versions if already installed.

Next, time to actually run the example playbook and manage our first Proxmox virtual machines:

```shell
ansible-playbook -i inventory manage_proxmox_vms.yml
```

To specify a particular user, use `-u root` (replacing `root`), and if you need to provide passwords, use `-k` for SSH password and/or `-K` for sudo password; in example:

```shell
ansible-playbook -i inventory manage_proxmox_vms.yml -K -u admin1
```

This will ask for a sudo password, then login to the `admin1` user (using public key auth - add `-k` for password) and run the playbook.

After the initial setup, the virtual machine states can be managed as group or individually by populating parameter `proxmox_vm_states_targets` with hostnames and calling the playbook with tag `proxmox_vm_manage_states`; in example:

```shell
ansible-playbook -i inventory manage_proxmox_vms.yml -K -u admin1 -t proxmox_vm_manage_states
```
