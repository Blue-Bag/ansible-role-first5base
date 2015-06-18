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

License
-------

BSD

Refs
------------------

http://www.richud.com/wiki/Ubuntu_Enable_Automatic_Updates_Unattended_Upgrades
