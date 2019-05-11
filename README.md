ansible-role-ssh-idempotent
===========================

Idempotent SSH port and user setup.

This role use the inventory information as target configuration but can also
detect if a "bootstrap" configuration need to be used.

Moreover with caching enabled, this role allows changing the ssh port of a host.
To do that:
 - run the role with caching enabled (to record the current port)
 - update the port in the inventory
 - run the role again, it will configure the new port (using the cached one)

Requirements
------------

The role still require a valid user with `become` privilege to setup the new user and port.

It's recommended to enable caching in your `ansible.cfg`:

    [defaults]
    #enable caching
    gathering = smart
    fact_caching = jsonfile
    fact_caching_connection = ../cachedir
    fact_caching_timeout = 86400

For more caching option see: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#fact-caching


Another requirement is to define the variable for user creation or to use the
hook to create the user yourself (see variables bellow).

Role Variables
--------------

The role use the inventory `ansible_port` and `ansible_user` as target configuration but will use `ssh_bootstrap_port` and `ssh_bootstrap_user` as fallback when the port/user is not yet configured.

`ansible_port`: used by the role as the target to configure the ssh port

`ansible_user`: user to be configured as connection user.
                default to empty, in that case don't configure the user.
                The empty string means let ssh configuration do it's job and
                connect as current user would (ie. use `.ssh/config`).


`ssh_bootstrap_port`: default to 22, port to try if `ansible_port` fail.

`ssh_bootstrap_user`: default to empty, user to try if `ansible_user` fail.
                      The empty string means let ssh configuration to it's job
                      and connect as current user would.

Before restarting the ssh server we must ensure both the detected port and
the new port are accessible (ie firewall, selinux, apparmor ... are correctly
configured)

`ssh_before_restart_hook`: list of absolute path to task list to include before
                           we restart the ssh server.
    The variable `ssh_port_list` will be defined with a list of ports to allow
    (two when the port changed and only one in other cases). At this stage,
    the requested port, user are in `ssh_configured_port`, `ssh_configured_user`    and the current connection port, user are in `ansible_port`, `ansible_user`
    (ie before update).

`ssh_do_configure_port`: default to true, when false the ssh configuration file
                         will not be updated to change the port, you must do it
                         within the hook.

`ssh_do_configure_user`: default to true, when false the user will not be
                         created, it must either already exists or you have
                         to create it within the hook.

`ssh_do_discover_port`: default to true, when false the ssh port will not be
                        detected. To use with `ssh_do_configure_port:false` to
                        skip port change (ie when using the default)

`ssh_do_discover_user`: default to true, when false the ssh user will not be
                        detected. To use with `ssh_do_configure_user:false` to
                        skip user change (ie when not adding one)

`ssh_deploy_user_password`: default UNDEFINED, to create the user you MUST
                            define this variable to the encrypted password
                            (unless the hook handles the user creation)

`ssh_deploy_user_uid`: default to 9999, the user's UID.

`ssh_deploy_user_shell`: default to `/bin/bash` the user's shell

`ssh_deploy_user_home`: default to `/home/<ansible_user>` the user's home
                        directory

`ssh_deploy_user_public_keys`: default UNDEFINED, to add ssh public keys place
                               them in this array. Example:

```
ssh_deploy_user_public_keys:
  - "{{ lookup('env','HOME') ~ '/.ssh/id_rsa.pub' }}"
```

`ssh_do_update_settings`: default to true, update the sshd configuration file.

`sshd_default_settings`: Set of configuration values to update in sshd config.

DO NOT USE THIS FOR THE PORT, USE `ansible_port` INSTEAD

```
sshd_default_settings:
  "PasswordAuthentication": "no"  # Disable password authentication
  "PermitRootLogin": "no"         # Disable SSH root login
  "PermitTunnel": "no"            # Disable tun(4) device forwarding

```

The 3 following variables are merged with `sshd_default_settings` to build the final setting applied to the host.

`sshd_settings` : This variable allows updating the default settings for all hosts.

`sshd_group_settings` : This variable should be set for a group to update the settings for that group.

`sshd_host_settings` : This variable should be set for a host to update the settings for that host.


Example Playbook
----------------

    - hosts: servers
      gather_facts: no
      roles:
         - { role: ansible-role-ssh-idempotent }

Example Inventory
-----------------

    [host]
    sample-host1 ansible_port=2222
    sample-host2 ansible_port=2222 ansible_user=deploy
    sample-host3 ansible_port=2222 ansible_user=deploy ssh_bootstrap_user=root


The current user must already have ssh connexion configured to sample-host1 and
sample-host2.

On sample-host1 the port will be updated from 22 to 2222.

On sample-host2 the port updated from 22 to 2222, and the 'deploy' user added,
next connexions will use the 'deploy' user.

sample-host3 will be connected as root (so the ssh public key must be configured
in the root account) then port updated from 22 to 2222, and the 'deploy' user
added, next connexions will use the 'deploy' user. (root ssh connexion can be
disabled after the role has run)

License
-------

Copyright 2018-2019 Julien Viard de Galbert

Licensed under the Apache License, Version 2.0 (the "License"); you may
not use this file except in compliance with the License. You may obtain
a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations
under the License.

Credits
-------

I want to credit the following sources that inspired me in writing this module:

- The implementation is based on the technique described in the blog post
[Changing the SSH port with Ansible](https://dmsimard.com/2016/03/15/changing-the-ssh-port-with-ansible/)
by David Moreau Simard <dms@redhat.com>.

- The hook idea comes from the article [Adding hooks to your Ansible roles](https://coderwall.com/p/arh7bq/adding-hooks-to-your-ansible-roles)
by Ramon de la Fuente.

:+1: **Thanks guys !**

Author Information
------------------

Julien Viard de Galbert

http://silicone.blogsite.org/
