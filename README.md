# RabbitMQ Ansible Role



## Role Name: RabbitMQ

An Ansible Role that installs RabbitMQ and configures clustering, plugins, vhosts, and users.

### Role Variables

#### `defaults/main.yml`

```
enable_rabbitmq_repo: True
install_rabbitmq_from_pkgs: False

rabbitmq_version: 3.7.6-1
erlang_version: 20.3.6-1
rabbitmq_nodename: "rabbit@{{ ansible_default_ipv4.address }}"

rabbitmq_cluster: True
rabbitmq_cluster_master: "rabbit@{{ hostvars[ansible_play_hosts.0].ansible_default_ipv4.address }}"
rabbitmq_erlang_cookie_file: /var/lib/rabbitmq/.erlang.cookie
rabbitmq_erlang_cookie: "{{ vault_rabbitmq_erlang_cookie }}"

# Plugins
rabbitmq_plugin_dir: "/usr/lib/rabbitmq/lib/rabbitmq_server-{{ rabbitmq_version.split('-').0 }}/plugins"

rabbitmq_plugins: []
rabbitmq_plugins_disabled: []

# Users

rabbitmq_users: []
rabbitmq_users_absent: []

# Vhosts

rabbitmq_vhosts: []
rabbitmq_vhosts_absent: []
```
### Example Group Variables Configuration
#### `group_vars/openstack-project/vars`
```
rabbitmq_erlang_cookie: "{{ vault_rabbitmq_erlang_cookie }}"
rabbitmq_cluster: True

install_rabbitmq_from_pkgs: True

rabbitmq_plugins:
  - rabbitmq_management

rabbitmq_users:
  - user: openstack
    password: "{{ vault_openstack_rabbit_pass }}"
    tags: administrator

rabbitmq_users_absent:
  - guest
```
#### `group_vars/openstack-project/vault`
```
vault_rabbitmq_erlang_cookie: ABCDEFGHIJKLMNOPQRST
vault_openstack_rabbit_pass: rabbitmqpass
```

### Configuring Plugins, Users, and VHosts

#### Plugins
```
+-----------------------------------------------------+
| parameter | required | default | choices | comments |
|:----------|:---------|:--------|:--------|:---------|
| name      | yes      |         |         |          |
| url       | no       |         |         |          |
+-----------------------------------------------------+

rabbitmq_plugins:
  - rabbitmq_management
  - name: rabbitmq_delayed_message_exchange
    url: http://www.rabbitmq.com/community-plugins/v3.6.x/rabbitmq_delayed_message_exchange-0.0.1.ez

rabbitmq_plugins_disabled: []
```
#### Users
```
+----------------------------------------------------------+
| parameter      | required | default | choices | comments |
|:---------------|:---------|:--------|:--------|:---------|
| configure_priv | no       | .*      |         |          |
| password       | yes      |         |         |          |
| read_priv      | no       | .*      |         |          |
| tags           | no       |         |         |          |
| user           | yes      |         |         |          |
| vhost          | no       | /       |         |          |
| write_priv     | no       | .*      |         |          |
+----------------------------------------------------------+

rabbitmq_users:
  - user: admin
    password: {{ vault_rabbitmq_admin_password }}
    tags: administrator
    vhost: /
    read_priv: .*
    write_priv: .*
    configure_priv: .*
  - user: monitor
    password: {{ vault_rabbitmq_monitor_password }}
    vhost: /
    read_priv: .*

rabbitmq_users_absent:
  - guest
```

#### Vhosts
```
+------------------------------------------------------------------------------+
| parameter | required | default | choices                          | comments |
|:----------|:---------|:--------|:---------------------------------|:---------|
| name      | yes      |         |                                  |          |
| node      | no       | rabbit  |                                  |          |
| tracing   | no       | no      | <ul><li>yes</li><li>no</li></ul> |          |
+------------------------------------------------------------------------------+

rabbitmq_vhosts:
  - /one
  - name: /two
    node: rabbit
    tracing: no

rabbitmq_vhosts_absent: []
```

### Example Playbook

I like to include my variables in `group_vars`, but you may wish to include them in your `playbook.yml` instead.

```
- name: Deploy RabbitMQ
  hosts: openstack-rabbitmq
  become: True
  gather_facts: True
  tasks:
    - include_role:
        name: rabbitmq
      vars:
        rabbitmq_erlang_cookie: "{{ vault_rabbitmq_erlang_cookie }}"
        rabbitmq_cluster: True
        rabbitmq_plugins:
          - rabbitmq_management 
        rabbitmq_users:
          - user: openstack
            password: "{{ vault_openstack_rabbit_pass }}"
            tags: administrator   
        rabbitmq_users_absent:
          - guest
```

### Running Playbook

```
ansible-playbook rabbitmq.yml
```