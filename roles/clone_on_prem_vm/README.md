clone_on_prem_vm
================

A role to clone an existing on prem VM using the KVM hypervisor. The role sets the **clone_on_prem_vm_local_image_path** variable containing the path where the image was saved on localhost. This role requires privilege escalation because the .qcow2 file created by ``virt-clone`` is owned by root and ``qemu-img convert`` requires access to convert it to .raw.

Requirements
------------

**qemu** and **qemu-img** packages installed.

Role Variables
--------------

* **clone_on_prem_vm_source_vm_name**: (Required) The name of the on-prem VM you want to clone.
* **clone_on_prem_vm_image_name**: The name you want to call the cloned image. If not set, the **clone_on_prem_vm_source_vm_name** will be used with a _-clone_ suffix.
* **clone_on_prem_vm_overwrite**: Weather to overwrite or not an already existing on prem VM clone. Default: true.
* **clone_on_prem_vm_local_image_path**: The path where you would like to save the image. If the path does not exists on localhost, the role will create it. If this parameter is not set, the role will save the image in a _~/tmp_ folder.
* **clone_on_prem_vm_uri**: Libvirt connection uri. Default: "qemu:///system".

Dependencies
------------

N/A

Example Playbook
----------------

    - hosts: localhost
      gather_facts: false

      vars:
        on_prem_source_vm_name: "ubuntu-guest"
        on_prem_vm_image_name: "ubuntu-guest-image"
        local_image_path: "~/images/"
        kvm_host:
          name: kvm
          ansible_host: 192.168.1.117
          ansible_user: vagrant
          ansible_ssh_private_key_file: ~/.ssh/id_rsa.pub

      tasks:
        - name: Add host to inventory
          ansible.builtin.add_host:
            name: "{{ kvm_host.name }}"
            ansible_host: "{{ kvm_host.ansible_host }}"
            ansible_user: "{{ kvm_host.ansible_user }}"
            ansible_ssh_common_args: -o "UserKnownHostsFile=/dev/null" -o StrictHostKeyChecking=no -i {{ kvm_host.ansible_ssh_private_key_file }}
            groups: "libvirt"

        - name: Import 'cloud.aws_ops.clone_on_prem_vm' role
          ansible.builtin.import_role:
            name: cloud.aws_ops.clone_on_prem_vm
          vars:
            clone_on_prem_vm_source_vm_name: "{{ on_prem_source_vm_name }}"
            clone_on_prem_vm_dest_image_name: "{{ on_prem_vm_image_name }}"
            clone_on_prem_vm_local_image_path: "{{ local_image_path }}"
          delegate_to: kvm

License
-------

GNU General Public License v3.0 or later

See [LICENCE](https://github.com/ansible-collections/cloud.aws_ops/blob/main/LICENSE) to see the full text.

Author Information
------------------

- Ansible Cloud Content Team
