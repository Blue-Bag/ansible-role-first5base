First5 Base
=========

This is the base role of the First5 set and covers the following tasks:
1. Update the package Manager Repository (apt)
2. Set up unattended updates (security)
3. Sets up the HOSTS
4. Create an Ansible Managed MOTD file

Big todo: Update for other package managers (RedHat)

Updated to respect diff b/w Debian and Ubuntu in 50unattended-upgrades

Requirements
------------
Note that the disabling of services is not done on local group
as this may disable NFS mounting if rpcbind is specified.
see http://backdrift.org/fixing-rpc-statd-running-required-remote-locking

If the rpc.statd service is stopped

etc/init.d/nfslock start

Another approach is to clear the default list of services to disable in the group_vars/local

srv_to_stop:

Hosts Files
--------------
The role enables the writing of hosts entries for all servers in a groups f5base_hosts_group
For this to work you need to have gatherd facts for all inventory, even in limiting the play.
To do that add the follwoing block to run before the play that runs this role
- hosts: servers
  tasks:
    - name: gather facts from servers
      setup:
      delegate_to: "{{item}}"
      delegate_facts: True
      loop: "{{ groups[f5base_hosts_group] }}"
      when: item not in groups['local']
  tags: ['always']

Role Variables
--------------


Dependencies
------------


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: f5-base , tags: ["base"] }

Tests
-----------------
At present the setting of hostnames is failing in the docker based tests (see https://github.com/ansible/ansible/issues/19681)



License
-------

BSD

Refs
------------------

http://www.richud.com/wiki/Ubuntu_Enable_Automatic_Updates_Unattended_Upgrades
Also see:
https://github.com/jnv/ansible-role-unattended-upgrades

setup VIM
#https://gist.github.com/jackbravo/7163461
#https://github.com/axai-mx/ansible-drupal-roles/blob/master/roles/common/tasks/main.yml
#http://superuser.com/questions/27091/vim-to-replace-vi

Locales:
https://andreas.scherbaum.la/blog/archives/941-Configuring-locales-in-Debian-and-Ubuntu,-using-Ansible-Reloaded.html
