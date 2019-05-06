TEST: ansible-role-ssh-idempotent
=================================

This file describe the manual step to test the role.

## Setup

- Create a host for testing (a vm, a container, an instance on a cloud provider...)
- Update the `inventory` file to configure your new host, add `ssh_bootstrap_port` and `ssh_bootstrap_user` if the default values don't work for you.
- Also update the `group_vars/all.yml` to set the ssh public key to use
(default is `~/.ssh/id_rsa.pub`)

*Note:* The default `ssh_bootstrap_user` is empty, so it will use your current
user. Alternatively you can to use your ~/.ssh/config to configure the user.

## Test1 : bootstrap

### Launch

    ansible-playbook -i inventory test.yml

### Results

It should:
 - connect to your host (using `ssh_boostrap_*`),
 - update it's ssh port to `2342`,
 - create a user `toto` with password `test` and uid `9999`
 - the task `Show ssh_port_list in hook` should display:
```
ok: [test_host] => {
    "ssh_port_list": [
        2342, 
        22
    ]
}
```

*Note:* You have to verify all that manually.

## Test2 : idempotent

### Launch

    ansible-playbook -i inventory test.yml

### Results

It should:
 - connect to your host using previously configured port and user,
 - the task `Show ssh_port_list in hook` should display:
```
ok: [test_host] => {
    "ssh_port_list": [
        2342
    ]
}
```
 - not change anything else.

## Test3 : port change

### Update

Manually update the `inventory` file and change the `ansible_port` value to 232.

### Launch

    ansible-playbook -i inventory test.yml

### Results

It should:
 - connect to your host (using `toto` and port `2342`),
 - update it's ssh port to `232`,
 - the task `Show ssh_port_list in hook` should display:
```
ok: [test_host] => {
    "ssh_port_list": [
        232, 
        2342
    ]
}
```
 - not change anything else.

## Test4 : unreachable

### Update

Shutdown your VM or update the `inventory` file and change the host to an inexisting one.

### Launch

    ansible-playbook -i inventory test.yml

### Results

It should:
 - try to connect to the configured and bootstrap ports (failure ignored)
 - fail with an unreachable error:
```
PLAY RECAP ****************************************************************************
test_host                  : ok=6    changed=0    unreachable=1    failed=0   
```

