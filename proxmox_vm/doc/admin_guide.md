# Administration guide for **proxmox_vm**

This file is intended for system engineers who modify the Ansible role [proxmox_vm](<url_to_be_provided>/roles/proxmox_vm).
*The engineer is required to have an advanced understanding about Ansible and to be able to create/modify Ansible roles.*

## Git

The entire Ansible implementation and life-cycle is subject to the [Git Workflow](<url_to_be_provided>/academy/ansible_guidelines/tree/master#git-workflow). All modifications have to follow the workflow in order to be more manageable, measurable and maintainable including a track record of changes. In addition to the workflow, [release management guidelines](<url_to_be_provided>/academy/ansible_guidelines/tree/master#release-management) are in affect for every type of release regarding the evolutionary cycle of the Ansible implementation.

## Editable Proxmox VM sections

A lot of stuff is going on when creating or configuring virtual machines in a Proxmox environment. This is why it is split up in sections.
Each section that is allowed for custom adjustments is explained here below synchronously to the sections in the [defaults/main.yml](../defaults/main.yml) file. In the [docs/user_guide.md](user_guide.md) examples will be provided on how to implement the variables.

### Remote repository

Variable `proxmox_vm_remote_repository` is set to the URL of the remote repository from which we download our external requirements like `proxmoxer` and cloud-init images.

```yaml
proxmox_vm_remote_repository: "<string>"
```

By default this variable is left emty.

### Proxmoxer package filename

Variable `proxmox_vm_pypi_proxmoxer` is set to the package filename, on the remote repository, for `proxmoxer` (i.e. `proxmoxer-1.2.0.tar.gz`).

```yaml
proxmox_vm_pypi_proxmoxer: "<string>"
```

By default this variable is left empty.  

### Templates management

Variable `proxmox_vm_templates_management` defines wether or not to manage the virtual machine templates.

```yaml
proxmox_vm_templates_management: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### Template image filename

Variable `proxmox_vm_template_remote_image` defines which extra packages to install (i.e. `rhel-8.5-x86_64-kvm.qcow2`).

```yaml
proxmox_vm_template_remote_image: "<string>"
```

By default this variable is left empty.

### Download remote cloud-init image

Variable `proxmox_vm_template_remote_image_download` defines wether or not to download the specified cloud-init image.

```yaml
proxmox_vm_template_remote_image_download: [ <true> | <false> ]
```

By default this variable is set to ```true```.

> The image download location will be compiled in `vars/main.yml` by variable ```proxmox_vm_template_remote_image_url```.

### Remote cloud-init image overwrite

Variable `proxmox_vm_template_remote_image_overwrite` defines wether or not to overwrite the pre-existing cloud-init image if the filename matches.

```yaml
proxmox_vm_template_remote_image_overwrite: [ <true> | <false> ]
```

By default this variable is set to ```false```.

### Remove pre-existing template

Variable `proxmox_vm_template_remove` defines wether or not to remove the pre-existing template.

```yaml
proxmox_vm_template_remove: [ <true> | <false> ]
```

By default this variable is set to ```false```.

### Template name

Variable `proxmox_vm_template_name` is used to define the name of the virtual machine template (i.e. `rhel85`).

```yaml
proxmox_vm_template_name: "<string>"
```

By default this variable is left empty.

### Template virtual machine id

Variable `proxmox_vm_template_vm_id` is used to define the ID of the virtual machine template. It is advisable to keep this in a higher range than your virtual machines will be.

```yaml
proxmox_vm_template_vm_id: "<integer>"
```

By default this variable is set to ```10000```.

### Template virtual machine memory

Variable `proxmox_vm_template_memory` defines the amount of memory for the virtual machine template.

```yaml
proxmox_vm_template_memory: "<integer>"
```

By default this variable is set to ```2048```.

### Proxmox hypervisor

Variable `proxmox_vm_hypervisor` is used to define the name of the primary node in the Proxmox VE cluster (i.e. `{{ groups['proxmox'][0] }}`).

```yaml
proxmox_vm_hypervisor: "<string>"
```

By default this variable is left empty.

### Proxmox hypervisor API endpoint

Variable `proxmox_vm_hypervisor_api` defines the location of the API endpoint of the Proxmox VE hypervisor (i.e. `{{ hostvars[groups['proxmox'][0]].ansible_default_ipv4.address }}:443`).

```yaml
proxmox_vm_hypervisor_api: "<string>"
```

By default this variable is left empty.

### Proxmox hypervisor user

Variable `proxmox_vm_hypervisor_user` is used to define a username with sufficient privileges in the Proxmox VE cluster (i.e. `admin`).

```yaml
proxmox_vm_hypervisor_user: "<string>"
```

By default this variable is left empty.

### Proxmox hypervisor user realm

Variable `proxmox_vm_hypervisor_user_realm` is used to define a realm to which the user can login to (i.e. `pam`).

```yaml
proxmox_vm_hypervisor_user_realm: "<string>"
```

By default this variable is left empty.


### Proxmox hypervisor user password

Variable `proxmox_vm_hypervisor_user_password` is used to define the users password (i.e. `mysupersecretpassword`).

> This value should be stored in a vault.

```yaml
proxmox_vm_hypervisor_user_password: "<string>"
```

By default this variable is left empty.

### Guest nodes

Variable `proxmox_vm_guest_nodes` is a list object which needs to contain the nodenames of the virtual machines (i.e. `{{ groups['virtualmachines'] }}`).

> It is advisable to use an inventory group as source, due to the fact this role needs host_vars to be able to configure virtual machines.

```yaml
proxmox_vm_guest_nodes: []
```

By default this variable is left empty.

### Cloud-init username

Variable `proxmox_vm_ci_username` is is used to define the cloud-init user for each create virtual machine (i.e. `ci-admin`).

```yaml
proxmox_vm_ci_username: "<string>"
```

By default this variable is left empty.

### Cloud-init password

Variable `proxmox_vm_ci_password` is is used to define the password of the cloud-init user for each create virtual machine.

> To use a password, ensure to provide a password with SHA512 (i.e. with `mkpasswd --method=SHA-512 --rounds=4096` )

```yaml
proxmox_vm_ci_password: "<string>"
```

By default this variable is left empty.

### Cloud-init SSH-keys

Variable `proxmox_vm_ci_ssh_keys` is a list object which can contain SSH-keys of the cloud-init user.

```yaml
proxmox_vm_ci_ssh_keys: []
```

By default this variable is left empty.

### Cloud-init enable password authenitcation

Variable `proxmox_vm_ci_enable_ssh_password_auth` is is used to define wether or not the cloud-init user will be able to login over SSH using a password.

```yaml
proxmox_vm_ci_enable_ssh_password_auth: "<string>"
```

By default this variable is set to ```false```.

### Guest storage

Variable `proxmox_vm_storage` is is used to define the virtual machine storage location on the hypervisor (i.e. `local-lvm`).

```yaml
proxmox_vm_storage: "<string>"
```

By default this variable is set to ```local-zfs```.

### Guest disk resize

Variable `proxmox_vm_resize_disk` is is used to define wether or not to resize the virtual machines disk to a specified size after cloning.

```yaml
proxmox_vm_resize_disk: [ <true> | <false> ]
```

By default this variable is set to ```false```.

### Qemu agent

Variable `proxmox_vm_enable_qemu_agent` is is used to define wether or not to enable the qemu agent on the virtual machine.

```yaml
proxmox_vm_enable_qemu_agent: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### DNS domain

Variable `proxmox_vm_dns_fqdn` is is used to define the DNS domain in which the virtual machine is registered (i.e. `example.com`).

```yaml
proxmox_vm_dns_fqdn: "<string>"
```

By default this variable is left empty.

### DNS servers

Variable `proxmox_vm_dns` is a list object which can contain IP addresses of the DNS servers for the virtual machine.

```yaml
proxmox_vm_dns: []
```

By default this variable is left empty.

### Extra VM packages

Variable `proxmox_vm_install_packages` is a list object which can contain packages which need to be installed in the virtual machine after creation.

```yaml
proxmox_vm_install_packages: []
```

By default this variable is left empty.

### Update VM after boot

Variable `proxmox_vm_enable_update_after_boot` is is used to define wether or not to update the virtual machines after the initial boot.

```yaml
proxmox_vm_enable_update_after_boot: [ <true> | <false> ]
```

By default this variable is set to ```false```.

### Run commands in VM

Variable `proxmox_vm_run_cmd` is a list object which can contain commands which need to be ran in the virtual machine after the intial boot.

```yaml
proxmox_vm_run_cmd: []
```

By default this variable is left empty.

### Local storage path

Variable `proxmox_vm_local_storage_path` is is used to define the local storage path on the hypervisor.

```yaml
proxmox_vm_dns_fqdn: "<string>"
```

By default this variable is set to ```/var/lib/vz```.

### Cloud-init image path

Variable `proxmox_vm_cloudinit_image_location` is is used to define the cloud-init images path on the hypervisor.

```yaml
proxmox_vm_cloudinit_image_location: "<string>"
```

By default this variable is set to ```{{ proxmox_vm_local_storage_path }}/template/iso```.

### Cloud-init snippets path

Variable `proxmox_vm_cloudinit_snippets_location` is is used to define the cloud-init snippets path on the hypervisor.

```yaml
proxmox_vm_cloudinit_snippets_location: "<string>"
```

By default this variable is set to ```{{ proxmox_vm_local_storage_path }}/snippets```.

### Cloud-init volume

Variable `proxmox_vm_cloudinit_volume` is is used to define the cloud-init volume on the hypervisor.

```yaml
proxmox_vm_cloudinit_volume: "<string>"
```

By default this variable is set to ```local```.

### Manage VM states

Variable `proxmox_vm_states_managed` is is used to define wether or not to manage the virtual machines state after creation.

```yaml
proxmox_vm_states_managed: [ <true> | <false> ]
```

By default this variable is set to ```true```.

### VM desired state

Variable `proxmox_vm_states_desired` is is used to define the state of the virtual machines after creation.

```yaml
proxmox_vm_states_desired: "<string>"
```

> This variable can be of values [started, stopped, restarted, absent].

By default this variable is set to ```started```.

### VM desired state targets

Variable `proxmox_vm_states_targets` is a list object which can contain target virtual machines for which the state must be managed.

> If left undefined or as empty list object, `proxmox_vm_guest_nodes` will have their states managed when `proxmox_vm_states_managed` is set to ```true```.

```yaml
proxmox_vm_states_targets: []
```

By default this variable is left empty.
