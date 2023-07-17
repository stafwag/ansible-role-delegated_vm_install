# Delegated VM install

An Ansible role to install a libvirt virtual machine with ```virt-install```
and ```cloud-init```. 

This role is designed to delegate the install to a libvirt hypervisor.

It allows you to specify the Linux/libvirt KVM host as part of the virtual machine definition.

# Installation

## Ansible galaxy

The role is available on [Ansible Galaxy](https://galaxy.ansible.com/stafwag/delegated_vm_install).

To install the role from Ansible Galaxy execute the command below. 

```bash
ansible-galaxy install stafwag.delegated_vm_install
```

## Source Code

If you want to use the source code directly.

Clone the role source code.

```bash
$ git clone https://github.com/stafwag/ansible-role-virt_install_vm
```

and put into the [role search path](https://docs.ansible.com/ansible/2.4/playbooks_reuse_roles.html#role-search-path)

## Requirements

### Supported GNU/Linux Distributions

It should work on most GNU/Linux distributions.
```cloud-localds``` is required. ```cloud-localds``` was available on
Centos/RedHat 7 but not on Redhat 8. You'll need to install it manually
to use the role on Centos/RedHat 8.

* Archlinux
* Debian
* Centos 7
* RedHat 7
* Ubuntu

### Roles

The role is a wrapper around the following roles:

  * **stafwag.libvirt**:
    [https://github.com/stafwag/ansible-role-libvirt](https://github.com/stafwag/ansible-role-libvirt)
  * **stafwag.qemu_img**:
    [https://github.com/stafwag/ansible-role-qemu_img](https://github.com/stafwag/ansible-role-qemu_img)
  * **stafwag.cloud_localds**:
    [https://github.com/stafwag/ansible-role-cloud_localds](https://github.com/stafwag/ansible-role-cloud_localds)
  * **stafwag.virt_install_import**:
    [https://github.com/stafwag/ansible-role-virt_install_import](https://github.com/stafwag/ansible-role-virt_install_import)
  * **stafwag.virt_install_vm**:
    [https://github.com/stafwag/ansible-role-virt_install_vm](https://github.com/stafwag/ansible-role-virt_install_vm)
  * **stafwag.etc_hosts**:
    [https://github.com/stafwag/ansible-role-etc_hosts](https://github.com/stafwag/ansible-role-etc_hosts)
  * **stafwag.package_update**:
    [https://github.com/stafwag/ansible-role-package_update](https://github.com/stafwag/ansible-role-package_update)

#### Ansible galaxy

When you installed the role with Ansible galaxy. The roles is installed in your ```ANSIBLE_ROLES_PATH```.
If not specified ```${HOME}/.ansible/roles/``` is used.

Go to the installed role

```bash
$ cd ~/.ansible/roles/stafwag.delegated_vm_install/
```

To install the required roles from Ansible galaxy execute

```bash
$ ansible-galaxy install -r requirements.yml
```

will install the latest version available on Ansible galaxy.

Or follow the installation instructions for each role on Ansible Galaxy.

[https://galaxy.ansible.com/stafwag](https://galaxy.ansible.com/stafwag)

#### Source (github)

To install the required role from source code execute.

```bash
$ ansible-galaxy install -r requirements.yml
```

this will install the latest default branch.


# Role Variables and templates

## Variables

### Virtual Machine specific

* **vm_ip_address**: Required. The virtual machine ip address.
* **vm_kvm_host**: Required. The hypervisor host the role will delegate the installation.

### Input

The ```delegated_vm_install``` hash is used for the defaults to set up the virtual machine.
If you need to overwrite for a virtual machine or host group, you can use the ```delegated_vm_install_overwrite``` hash.
Both hashes will be merged, if a variable is in both hashes the values of ```delegated_vm_install_overwrite``` are used.


* **delegated_vm_install**:
    * **security**: Optional
        * **file**:

          The file permissions of the created images.
          * **owner**: user Default: ```root```
          * **group**: Default: ```kvm```
          * **mode**: Default: ```660```
        * **dir**:

          The directory permissions of the created images.
          * **owner**: Default: ```root```
          * **group**: Default: ```root```
          * **mode**: Default: ```0751```
    * **post**:

      Post actions on the created virtual machines. By default, the post actions are only executed when the virtual 
      machine is created by the role. If the virtual machine already exists the post actions are skipped, unless 
      **always** is set to ```true```.

      The ssh host key is not updated by default. If you set ***update_ssh_known_hosts*** to ```true```, the ssh host
      key of the virtual machine is updated to ```ansible_host``` in ```${HOME}/.ssh/known_hosts``` on the ansible host.

        * **pause**: Optional. Default: ```seconds: 10``` Time to wait before executing the post actions.
        * **update_etc_hosts** Optional. Default: ```true```
        * **ensure_running** Optional. Default: ```true```
        * **ensure_running** Optional. Default: ```true```
        * **package_update** Optional. Default: ```true```
        * **reboot_after_update** Optional. Default: ```true```
        * **always** Optional. Default: ```false```
        * **update_ssh_known_hosts** Optional. Default: ```false```

    * **vm**:

      The virtual machine settings.

      * **template**: Optional. Default. ```templates/vms/debian/12/debian_vm_template.yml``` The virtual machine template.
      * **hostname**: Optional. Default: ```{{ inventory_hostname }}```. The vm hostname.
      * **path**: Optional. Default: ```/var/lib/libvirt/images/```
      * **boot_disk**: Required
          **src**: The path to installation image.
      * **size**: Optional. Default: ```100G ```
      * **disks** Optional array with [stafwag.qemu_img](https://github.com/stafwag/ansible-role-qemu_img) disks.
      * **default_user**: 
        * **name**: Default: ```{{ ansible user }}``` if ```{{ ansible_user }}``` is not defined the environment variabele ```USER``` is used.
        * **passwd** : Default: ```false```. Set it to ```false``` to lock the user.
        * **ssh_authorized_keys**: The ssh keys default ```""```
      * **dns_nameservers:**: Optional. Default: ```9.9.9.9```
      * **dns_search**: Optional. Default ```''```
      * **interface**: Optional. Default ```enp1s0```
      * **gateway**: Required. The vm gateway.
      * **reboot**: Optional. Default: false. Reboot the vm after cloud-init is completed
      * **poweroff**: Optional. Default: true. Poweroff The vm after cloud-init is completed
      * **wait:** Optional. Default: ```-1```. The wait argument passed to ```virt-install```. A negative value will wait till The vm is shutdown. ```0``` will execut the ```virt-install``` command and disconnect
      * **commands:** Optional. Default ```[]```. List of commands to be executed duing the ```cloud-init``` phase.


* **delegated_vm_install_overwrite**:

    The overwrite hash. This allows you to overwrite the above settings. Can be useful to have specific global settings,
    and overwrite values for a virtual machine.

### Return variables

* **vm**:
  * **hostname**: The virtual machine hostname.
  * **path**: The virtual machine path.
  * **default_user**: The default user hash.
  * **dns_nameservers:**: Optional. Default: ```9.9.9.9```
  * **dns_search**: Optional. Default ```''```
  * **interface**: Optional. Default ```enp1s0```
  * **vm.ip_adress***: Required. The vm ip address.
  * **vm.ip_prefix**: Optional. Default ```24```
  * **vm.gateway**: Required. The vm gateway.
  * **qemu_img**: An array the qemu_img disks
  * **virt_install_import.disks**:  Disk string array. Used by 

# Example playbook

## Inventory

Create a inventory.

* inventory_vms.yml

```yaml
---
k3s_cluster:
  vars:
    ansible_user: ansible
    ansible_python_interpreter: /usr/bin/python3
    delegated_vm_install:
      vm:
        path: /var/lib/libvirt/images/k3s/
        boot_disk:
          src:
            /Downloads/isos/debian/amd64/11/debian-11-generic-amd64.qcow2
          size: 50G
        disks: 
          - name: "{{ inventory_hostname }}_zfsdata.qcow2"
            size: 20G
            owner: root
        memory: 4096
        cpus: 4
        dns_nameservers: 9.9.9.9
        gateway: 192.168.122.1
        default_user:
          passwd: false
          ssh_authorized_keys:
            - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
  hosts:
    k3s-master001:
      ansible_host: 192.168.122.20
      vm_ip_address: "{{ ansible_host }}"
      vm_kvm_host: vicky
    k3s-master002:
      ansible_host: 192.168.122.21
      vm_ip_address: "{{ ansible_host }}"
      vm_kvm_host: vicky
    k3s-master003:
      ansible_host: 192.168.122.22
      vm_ip_address: "{{ ansible_host }}"
      vm_kvm_host: vicky
```

## Playbook

* create_vms.yml

```yaml
---
- name: Setup the virtual machine
  hosts: k3s_cluster
  become: true
  gather_facts: false
  roles:
    - role: stafwag.delegated_vm_install
      vars:
        ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
```

## Run the playbook

```yaml
$ ansible-playbook -i inventory_vms.yml create_vms.yml
```

***Have fun!***
