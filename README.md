Ansible Role - Common/Baseline Configuration
=========

This role will set up a "common" baseline configuration. It is intended to be used before running more complex roles against a node.


It has been tested on Ubuntu Server, and Fedora Server + IoT.

Requirements
------------

TBD


Role Variables
--------------

You may want to override the following:
```
common_dns_resolver: "1.1.1.2"

common_timezone: "America/New_York"
# Update the URI to the latest version of docker-compose

common_docker_compose_uri: "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-linux-x86_64"

common_users:
  - name: "YourUsername"
    ssh_key: "ssh-ed25519 AAAAC3N..."
```

Dependencies
------------

TBD 

Example Playbook
----------------

```
# playbook.yml
---
- name: Set up baseline configuration
  hosts: all
  become: true
  roles:
    - role: ansible_common
    - role: any_other_roles
  vars:
    common_dns_resolver: "8.8.8.8" # Google
    common_users:
      - name: "YourUsername"
        ssh_key: "ssh-ed25519 AAAAC3N..."
```

License
-------

MIT

