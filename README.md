Ansible Role: pve_ct_deploy
=========

This repository contains Ansible playbooks and tasks to automate the deployment of Proxmox VE (PVE) containers. The automation process orchestrates several key steps, including prerequisite checks, OS template management, container creation, and container lifecycle management (start/stop/delete). The tasks are modular and tagged for flexible execution, ensuring compatibility across various environments.

```
    git clone https://github.com/lorephoenix/ansible-role-pve_ct_deploy pve_ct_deploy
```

Requirements
------------

- Ansible 2.11 or higher
- Proxmox VE 8.x or higher
- `community.general` collection (version 9.4.0 or higher)
- Valid Proxmox API credentials (user, password and token)
- The `new_lxc` variable should be populated with a list of container specifications, such as `hostname`, `vmid`, `memory`, `cores`, etc.

Role Variables
--------------

The following variables can be customized to suit your environment. Default values are defined in `defaults/main.yml`

### Proxmox Connection Details

| Variable | Value | Data Type | Required | Description |
| :--- | :--- | :--- | :--- | :--- |
| `pve_api_password`    | `undefined`               | String  | Mandatory | Password for Proxmox API authentication.       |
| `pve_host`            | `proxmox.example.com`     | String  | Mandatory | Proxmox host address.                          |
| `pve_port`            | `8006`                    | Integer | Optional  | Proxmox API port.                              |
| `pve_tokenid`         | `root@pam!mytokenid`      | String  | Mandatory | API token ID for authentication.               |
| `pve_token_secret `   | `undefined`               | String  | Mandatory | API secret token                               |
| `pve_validate_certs`  | `false`                   | Boolean | Optional  | Whether to validate SSL certificates.          |
| `pve_node`            | `proxmox`                 | String  | Mandatory | Proxmox node where containers will be created. |


### Container Deployment Variables

| Variable | Value | Data Type | Required | Description |
| :--- | :--- | :--- | :--- | :--- |
| `instance_state`      | `present`     | String                | Mandatory | Desired state of the instance (`present` or `absent`).| 
| `new_lxc`             | `[]`          | List of Dictionaries  | Mandatory | List of container definitions with their properties.  |  
| `pubkey`              | `undefined`   | String                | Optional  | SSH public key to be added to the container.          |
| `root_password`       | `undefined`   | String                | Mandatory | Root password for the container.                      |


Each container in `new_lxc` can include the following keys:
- `hostname`: (Required) The hostname for the container.
- `cores`: Number of CPU cores allocated.
- `memory`: Memory allocation in MB.
- `disk`: Disk size in GB.
- `swap`: Swap memory allocation in MB.
- `description`: A description for the container.
- `vmid`: VM ID for the container.
- `net0`: Network configuration string.
- `tags`: List of tags.
- `features`: List of additional features to enable.
- `nameserver`: Custom DNS nameserver configuration.
- `searchdomain`: Sets DNS search domain for a container.
- `onboot`: Specifies whether a VM will be started during system bootup.
- `unprivileged`: Indicate if the container should be unprivileged.
- `timezone`: Timezone used by the container.
- `hookscript_volid`: Optional hook script volume ID.


### Template and Storage Variables

| Variable | Value | Data Type | Required | Description |
| :--- | :--- | :--- | :--- | :--- |
| `pve_package`         | `debian-11`   | String  | Mandatory | String to match for retrieving the OS template.      |
| `pve_storage`         | `local-lvm`   | String  | Mandatory | Proxmox storage used for container root filesystems. |
| `pve_template_state`  | `present`     | String  | Mandatory | State for the template (present or absent).          |
| `pve_template_storage`| `local-lvm`   | String  | Mandatory | Target storage for the template.                     |


Dependencies
------------

This role makes use of the community.general collection, which is part of the Ansible package and includes many modules and plugins supported by Ansible community which are not part of more specialized community collections.

To install the collection:
```
    ansible-galaxy collection install -r requirements.yml
```

Example Playbook
----------------

Here’s an example of how to use this role:

```yaml
- name: (playbook) | Proxmox deploy containers
  hosts: localhost
  vars_files:
    - vault.yml  # Load encrypted file with Proxmox token
  roles:
    - role: pve_ct_download
      vars:
        # Proxmox Connection 
        pve_host: "pve1.example.com"
        pve_node: "pve1"
        pve_state: "present"
        pve_tokenid: "root@pam!Ansible"

        # Template and Storage Variables
        pve_package: "debian-12"
        pve_storage: "local-lvm"
        pve_template_state: "present"
        pve_template_storage: "local-lvm"

        # Container Deployment Variables
        instance_state: "present"
        hookscript_volid: "local:snippets/pve-hookscript-ansible_proxy.sh"
        new_lxc:
            - hostname: "lxc-1"
              disk: 16
              memory: 1024
              nameserver: "8.8.8.8 8.8.4.4"
              net0: "name=eth0,bridge=vmbr0,firewall=1,ip=192.168.1.64/24,gw=192.168.1.1"
              onboot: true
              searchdomains: "example.com"
              swap: 1024
              timezone: "Europe/Brussels"
            - hostname: "lxc-2"
              memory: 512
              nameserver: "8.8.8.8 8.8.4.4"
              net0: "name=eth0,bridge=vmbr0,firewall=1,ip=192.168.1.65/24,gw=192.168.1.1"
              onboot: true
              searchdomains: "example.com"
              timezone: "Europe/Paris"

```

License
-------

MIT

Author Information
------------------

- Christophe Vermeren | [GitHub](https://github.com/lorephoenix) | [Facebook](https://www.facebook.com/cvermeren)
