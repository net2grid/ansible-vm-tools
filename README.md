# Ansible Role: Packer RHEL/CentOS Configuration for Vagrant VirtualBox

This role configures RHEL/CentOS (minimal install) in preparation for it to be packaged as part of a .box file for Vagrant/VirtualBox deployment using [Packer](http://www.packer.io/).

## Requirements

Prior to running this role via Packer, you need to make sure Ansible is installed via a shell provisioner, and that preliminary VM configuration (like adding a vagrant user to the appropriate group and the sudoers file) is complete, generally by using a Kickstart installation file (e.g. `kickstart.cfg`) with Packer. 

An example array of provisioners for your Packer .json template would be something like:

	"provisioners": [{
			"type": "shell",
			"execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",
			"script": "scripts/ansible.sh"
		},
		{
			"type": "ansible-local",
			"playbook_file": "ansible/main.yml",
			"galaxy_file": "requirements.yml"
		},
		{
			"type": "shell",
			"execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",
			"script": "scripts/cleanup.sh"
		}
	],

The files should contain, at a minimum:

**scripts/ansible.sh**:

    #!/bin/bash -eux
    # Add the EPEL repository, and install Ansible.
    rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    yum -y install ansible python-setuptools

**ansible/main.yml**:

    ---
    - hosts: all
      become: yes
      gather_facts: yes
      roles:
        - net2grid.vm-tools

You might also want to add another shell provisioner to run cleanup, erasing free space using `dd`, but this is not required (it will just save a little disk space in the Packer-produced .box file).

If you'd like to add additional roles, make sure you add them to the `role_paths` array in the template .json file, and then you can include them in `main.yml` as you normally would. The Ansible configuration will be run over a local connection from within the Linux environment, so all relevant files need to be copied over to the VM; configuratin for this is in the template .json file. Read more: [Ansible Local Provisioner](http://www.packer.io/docs/provisioners/ansible-local.html).

## Role Variables

None.

## Dependencies

None.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: all
      roles:
         - { role: net2grid.vm-tools }

License
-------

BSD

Author Information
------------------

This role is based on code by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
