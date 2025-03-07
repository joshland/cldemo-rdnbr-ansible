Redistribute Neighbor Demo
===========================

This demo shows a topology using 'redistribute-neighbor' to add host reachability directly into a BGP routed fabric. Switches are Cumulus Linux and servers running Ubuntu. This playbook initialized a CLOS topology running BGP unnumbered in the fabric with numbered links towards the hosts, and installs a webserver on one of the hosts to serve as a Hello World. The playbook then configures Redistribute Neighbor (Rd-Nbr). Rd-Nbr pushes /32 host routes into the routing table when ARP messages are received from the hosts. When the demo runs successfully, any server on the network should be able to access the webserver (on server04) via the BGP routes established over the fabric.

This demo is written for the [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant) reference topology and applies the reference BGP unnumbered configuration from [cldemo-config-routing](https://github.com/cumulusnetworks/cldemo-config-routing).


Quickstart: Run the demo
------------------------
Before running this demo, install [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds) and [Vagrant](https://releases.hashicorp.com/vagrant/). The currently supported versions of VirtualBox and Vagrant can be found on the [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant).

    ### Bring up the vagrant topology
    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 leaf03 leaf04 spine01 spine02 server01 server02 server03 server04 edge01 edge02 internet
    
    ### setup oob mgmt server
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    sudo apt-get install software-properties-common -y
    sudo apt-add-repository ppa:ansible/ansible -y
    sudo apt-get update
    sudo apt-get install ansible -qy

    ### Run the rdnbr demo
    git clone https://github.com/cnidus/cldemo-rdnbr-ansible
    cd cldemo-rdnbr-ansible
    ansible-playbook run-demo.yml
    
    ### create traffic so that ARP entries are created on leaves for redistribute neighbour to work
    ssh server01
    ping 10.0.1.104
    
    ssh server04
    ping 10.0.1.101

    ### check reachability of server04 from server01
    ssh server01
    wget 10.0.1.104
    cat index.html


Topology
--------
This demo runs on a spine-leaf topology with four dual-attached hosts. Each device's management interface is connected to an out-of-band management switch and bridged with the out-of-band management server from which we run Ansible.


Setting up the Infrastructure
-----------------------------
Ansible is an extremely lightweight automation solution. Ansible does not require agents or software to be installed on the nodes to be managed, and does not run a daemon on the server. As long as key-based authentication and passwordless sudo is enabled on all of the devices for the management user, Ansible can be used for automation. Ansible playbooks can either be run manually by an admin to deploy changes as they are implemented, or they can be scheduled to automatically run via a cronjob. There is also a premium edition of Ansible available that further enables automation and administration of the network by introducing a web UI. 


Anatomy of an Ansible Playbook
------------------------------
### `hosts`
The *inventory* is a list of all of the hostnames in the network, broken out into various groups. In this playbook, we have a group for spines, leafs, and servers, but you can break them out into logical groups as well, such as `database_cluster` or `datacenter2_spines`. A host can be in multiple groups. All of the servers are automatically placed into the special group `all`.

### `group_vars/`
This folder contains variables applied across groups of servers, written in yaml format. We use variables in the `all` group to define our entire layer 3 fabric and leave it up to the templates to select the appropriate variables. In a more complicated infrastructure, we can create group-specific variable files to spread out the 

### `roles/`
This contains the tasks that can be applied to host groups. Each role contains up to four folders. In this demo, we deploy the quagga and ifupdown2 roles on our switches and servers, ifupdown on both servers, and apache only on server02.

 * `tasks/main.yml` - list of actual tasks to be applied
 * `handlers/main.yml` - list of handlers to be called when tasks result in a change (such as restarting daemons when config changes)
 * `templates/` - configuration file templates, written using Jinja2
 * `files/` - files to be copied to target hosts without modification

### `run-demo.yml`
The top-level playbook is the entry point for ansible. You can make multiple top-level playbooks, and run them seperately using the `ansible-playbook` command. Each top level playbook should specify a list of host groups, the user to login with, and the roles to be applied to these groups.


Expanding the Network
---------------------
To add a new device, add the new hostnames to the `hosts` file and add the appropriate variables to connect them with the rest of the network. In a well-written playbook, templates and tasks should not need to be changed when new devices are added to the infrastructure. If you are using the reference topology, you can run `vagrant up leaf03 server03` to add a third host and tor to the infrastructure.
