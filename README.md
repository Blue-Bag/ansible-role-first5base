
# First5 Base

This is the base role of the First5 set and covers the following tasks:

 1. Update the package Manager Repository (apt)
 2. Set up unattended updates (security)
 3. Sets up the HOSTS
 4. Sets up logrotation for the syslog
 5. Create an Ansible Managed MOTD file

Big todo: Update for other package managers (RedHat)
Updated to respect diff b/w Debian and Ubuntu in 50unattended-upgrades

## Requirements

Note that the disabling of services is not done on local group

as this may disable NFS mounting if rpcbind is specified.

see http://backdrift.org/fixing-rpc-statd-running-required-remote-locking
If the rpc.statd service is stopped
etc/init.d/nfslock start
Another approach is to clear the default list of services to disable in the group_vars/local
`srv_to_stop:`

## Hosts Files

The role enables the writing of hosts entries for all servers in a groups f5base_hosts_group

For this to work you need to have gathered facts for all inventory, even in limiting the play.

To do that add the following block to run before the play that runs this role

```
- hosts: servers
  tasks:
    - name: gather facts from servers
      setup:
      delegate_to: "{{item}}"
      delegate_facts: True
      loop: "{{ groups[f5base_hosts_group] }}"
      when: item not in groups['local']
  tags: ['always']
```
 ## Logrotate:

Sets up the logrotate for rsyslog

Allows for the following:

- gpg encryption of rotated logs

- custom script actions for post rotate and last action


## Role Variables
` defaults/main.yml`

#### Set the default for the UMASK
A more secure umask of 027 could be set
But this can have implications for things like
pulling changes via git.
if you go with a more restrictive umask then set a more lenient
one in the .bashrc/.bash_profile for that user

```
   default_umask: '022'

   f5base_umask_files:
     - { path: '/etc/profile', element: 'umask'}
     - { path: '/etc/login.defs', element: 'UMASK'}
     - { path: '/etc/init.d/rc', element: 'umask' }
```

#### Optional tasks
##### Set the hostname
```
   f5base_dohostname: true
   hostname_fqdn: "{{ ansible_fqdn }}"
   hostname_name: "{{ hostname_fqdn.split('.').0 }}"
```
##### Do some base security
```
   f5base_dosecurity: true
```
#####  Set the MOTD
```
   f5base_domotd: true
```
##### Update locales
```
  f5base_dolocale: yes

  f5base_locales:
    - "en_GB.UTF-8"
    - "en_US.UTF-8"
```
  #### Name Server Update

```
   nameserver_update: False
# These are set to OpenDNS / google

  f5base_nameserver_ips:
   - 208.67.220.220
   - 8.8.4.4
   - 208.67.222.222
   - 8.8.8.8
```
# Packages
#### Services to stop
```
   srv_to_stop:
    - rpcbind
```
#### Packages to install
```
   f5base_packages_required:
     - rsync
     - vim
     - patch
     - dbus
     - debsums
     - needrestart
     - locales
```
#### Unattended upgrade packages to install
```
   f5base_unattended_up_packages:
     - { name: 'unattended-upgrades', state: 'present'}
     - { name: 'apt-listchanges', state: 'present'}
     - { name: 'update-notifier', state: 'present'}
```
#### Packages to start
```
   f5base_packages_started:
    - rsync
```
  #### Packages to remove
```
   f5base_packages_remove:
    - landscape-client
    - landscape-common
    - whoopsie
```
### defaults for Apt Periodic
#### Enable the update/upgrade script (0=disable)

```
   apt_periodic_enable: "1"
```

#### Do "apt-get update" automatically every n-days (0=disable)

```
   apt_periodic_update: "1"
```
####  Do "apt-get upgrade --download-only" every n-days (0=disable)

```
   apt_periodic_dload_packages: "1"
   apt_notify_email: 'root'
   apt_notify_onlyonerror: 'true'
```

#### Run the "unattended-upgrade" security upgrade script every n-days (0=disabled)
Requires the package "unattended-upgrades" and will write a log in /var/log/unattended-upgrades
```
   apt_periodic_unattended_upgrade: "0"
```
#### Do "apt-get autoclean" every n-days (0=disable)
```
apt_periodic_autoclean: "7"
```
#### Automatically reboot if required?
```
   apt_automatic_reboot: 'False'
```
#### What packages to check
```
   apt_origin_patterns:
   - "origin=Debian,archive=stable,label=Debian-Security"
   - "origin=Debian,archive=oldstable,label=Debian-Security"
```
#### Allowed origins
```
   apt_allowed_origins:
    - "${distro_id}:${distro_codename}-security"
    - "${distro_id}ESM:${distro_codename}"
  # - "${distro_id}:${distro_codename}-updates"
  # - "${distro_id}:${distro_codename}-proposed"
  # - "${distro_id}:${distro_codename}-backports"
```

### Adding custom entries to the hosts file
```
   f5base_custom_hosts_entries: []
```
You can specific a group in your inventory to add to the hosts file
```
   f5base_hosts_group: 'servers'
```

### Logrotate for rsyslog
```
   f5base_dologrotate: no
   f5base_logrotate_rotate_number: 7
   f5base_logrotate_period: 'daily'
```
#### if putting files off to s3 then don't delay compress
```
   f5base_logrotate_delaycompress: yes
```
#### if putting files off to S3 then uses date names
```
   f5base_logrotate_datenames: no
   f5base_logrotate_dateformat: '-%Y-%m-%d'
```
You can specify actions for the prerotate, postrotate and/or lastaction script blocks
The command can be a command or a file name.
If it is a filename then specify the path.
Also if it is a file the role will put the destination file up using a template specified in the source.
```
f5base_logrotate_actions:
  - name: 'push to S3' - Arbritary
    type: 'lastaction' - Action block for the action [prerotate | postrotate | lastaction]
    cmd: 's3put-logfiles.sh' - Command or file name to call
    isfile: yes - If a file then out the file up
    file_dest: '/usr/local/bin' - Destination path for the file
    file_src: '/path/to/srcfile' - Source path/filename for the file
    cmd_param: '"$1 {{ ansible_fqdn }}/logs"' - params to pass to the command / file
```

**note:** can use %1 for the name of the rotated file

If 'shared scripts' is set then the fullset of names gets passed and the script gets called once
If not then the script gets called once for each rotated log

#### Note that the postrotate gets the log file name not the resulting rotated file name

### Encrypt the rotated logs
By default compression will be gzip.
You can specify GPG for encryption
```
   f5base_do_gpg: yes (default no)
   f5base_gpg_public_key_path: '/path/to/public_key'
   f5base_gpg_identity: 'ansible@youremailaddress.com'
```
#todo
Generalise the encryption method
Allow other methods

## Dependencies

None

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```
- hosts: servers
  roles:
    - { role: f5-base , tags: ["base"] }

```

## Tests
At present the setting of hostnames is failing in the docker based tests (see https://github.com/ansible/ansible/issues/19681)



## License

BSD


## Refs

http://www.richud.com/wiki/Ubuntu_Enable_Automatic_Updates_Unattended_Upgrades

Also see:

https://github.com/jnv/ansible-role-unattended-upgrades

### setup VIM

 - https://gist.github.com/jackbravo/7163461

 - https://github.com/axai-mx/ansible-drupal-roles/blob/master/roles/common/tasks/main.yml

 - http://superuser.com/questions/27091/vim-to-replace-vi

### Locales:

https://andreas.scherbaum.la/blog/archives/941-Configuring-locales-in-Debian-and-Ubuntu,-using-Ansible-Reloaded.html
