# Vagrant for DevStack

This projcet holds a Vagrantfile and ansible playbooks, which set up a virtual
machine, and run devstack on it.

The goal is, with a single vagrant up command, to fire up a virtual machine,
install all the requirements including openstack projects and review patches,
and then run devstack.

## Instructions

1. Clone the project.
2. Set up a libvirt server
3. (Optional) Install an SSH public key in common/keys/id_rsa.pub
4. Create a local.conf file (Or use some of the examples in common/local.confs)
5. Add (or update) a machine in directory.conf.yml (See configuration below)
6. Run: `vagrant up <machine name>` or `vagrant up` to start both controller and compute

Once the VM is set up, it can be accessed using `vagrant ssh -p --
-l stack`.  Other SSH options may follow if desired. devstack will be
running within a tmux session. To see it, use `tmux attach`.

## Configuration

Virtual machines are defined in `directory.conf.yml`. This is a yaml file with
the following structure:


```
machines:
- name: Virtual machine 1
  hypervisor: ...
  ... (See below)
- name: Virtual machine 1
  hypervisor: ...
  ... (See below)
```

Where a virtual machine is defined with the following properties:

| Field             | Description                            |
| ----------------- | -------------------------------------- |
| name              | VM name                                |
| hypervisor        | libvirt server object                  |
| memory            | VM memory                              |
| vcpus             | Number of virtual CPUs                 |
| box               | Name of Vagrant box to run.            |
| local_conf_file   | The local.conf file to take.           |
| projects          | objects describing installed projects  |
| devstack_version  | The devstack version to use            |

The projects objects have the following fields:

| Field    | Description                                                       |
| -------- | ----------------------------------------------------------------- |
| name     | The project name (e.g. dragonflow, neutron)                       |
| reviews  | A list of gerrit review numbers to cherry-pick (Optional)         |
| repo     | The repo from which to git clone (Optional)                       |
| version  | The git version to take (branch, tag, or commit hash) (Optional)  |

A `openvswitch` named project pointing to an openvswitch repository can be
added to build and install from it.

The hypervisor object has two fields:

| Field     | Description                                  |
| --------- | -------------------------------------------- |
| name      | The server name (will be looked up via DNS)  |
| username  | The user with which to log in                |

Sample configuration `directory.conf.yml.sample` can be used as a startingi
point.

## Customisation

The virtual machine is provisioned using the `devstack.yml` ansible playbook.
The playbooks used are available in the playbooks folder.


## Install Vagrant in opensuse and vagrant-libvirt

These steps describe how to install Vagrant for opensuse Leap 15.0

1 - Add the vagrant repo:
zypper addrepo https://download.opensuse.org/repositories/home:/pavlix:/Development/openSUSE_Leap_15.0/home:pavlix:Development.repo

2 - Install vagrant:
sudo zypper in vagrant

3 - Install libvirt-devel, ruby-devel and gcc:
sudo zypper -n in libvirt-devel ruby-devel gcc

4 - Install the vagrant-libvirt plugin:
vagrant plugin install vagrant-libvirt
<<<<<<< HEAD

5 - Add the user to the libvirt group
sudo usermod -a -G libvirt user_name

6 - Start libirt daemon
systemctl enable libvirtd
systemctl start libvirtd

7 - Install ansible
sudo zypper install ansible


## Start using openstack

Once the compute is up, remember to discover it by executing in the controller:

vagrant ssh controller
./home/stack/devstack/tools/discover_hosts.sh

Then, to start using openstack:

source /home/stack/devstack/openrc admin admin
