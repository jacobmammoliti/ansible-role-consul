[![Build Status](https://travis-ci.org/arctiqjacob/ansible-role-vault.svg?branch=master)](https://travis-ci.org/arctiqjacob/ansible-role-vault)

Ansible Role: Consul
===================

A role to deploy a production grade [HashiCorp Consul](https://www.consul.io/).

Requirements
------------

- Compute Admin role in a GCP Project

Role Variables
--------------

Ansible variables are listed below, along with default values (see `defaults/main.yml`):

Controls whether a separate account is created or not and what the user and group should be named.
```yaml
consul_user: 'consul'
consul_group: 'consul'
consul_create_account: true
```

Where to initialize Consul's home, data, and install directory.
```yaml
consul_home_directory: '/etc/consul.d'
consul_data_directory: '/opt/consul'
consul_install_directory: '{{ consul_data_directory }}/bin'
consul_config_file: '{{ consul_home_directory }}/consul.hcl'
consul_server_file: '{{ consul_home_directory }}/server.hcl'
```

The version of Consul to install and where it should download its binary from.
```yaml
consul_version: '1.4.0'
consul_archive: 'consul_{{ consul_version }}_linux_amd64.zip'
consul_download: 'https://releases.hashicorp.com/consul/{{ consul_version }}/{{ consul_archive }}'
```

Controls Consul agent behaviour.
```yaml
consul_datacenter: 'Arctiq'
consul_raft_multiplier: 1
consul_ui: true
consul_client_addr: '0.0.0.0'
```

Dependencies
------------

- None.

Example
-------------------------------

The following example deploys a three node Consul 1.7.2 cluster in Google Cloud Platform.

Create three compute instances which will host the Vault servers:
```shell
for i in 0 1 2; do
  gcloud compute instances create consul-${i} \
    --async \
    --no-address \
    --boot-disk-size 100GB \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --scopes=compute-ro \
    --tags consul
done
```

Create an inventory file:
```shell
cat > inventory <<EOF
[vault]
consul-0.c.[PROJECT_ID].internal
consul-1.c.[PROJECT_ID].internal
consul-2.c.[PROJECT_ID].internal
EOF
```

Create an Ansible playbook, calling the role:
```shell
cat > main.yaml <<EOF
---
- hosts: servers, clients
  become: yes
  roles:
    - role: ansible-role-consul
EOF
```

Ensure Python is installed on the servers:
```shell
ansible -i inventory vault -m ping
consul-0.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
consul-2.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
consul-1.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

Run the Ansible playbook:
```shell
$ ansible-playbook -i inventory main.yaml
...
PLAY RECAP *********************************************************************
consul-0.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0   
consul-1.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0   
consul-2.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0  
```

View Consul cluster:
```shell
$ export CONSUL_HTTP_ADDR=http://consul-0.c.[PROJECT_ID].internal:8500

$ consul members
Node      Address             Status  Type    Build  Protocol  DC      Segment
consul-0  10.128.0.7:8301     alive   server  1.7.1  2         arctiq  <all>
consul-1  10.128.0.12:8301    alive   server  1.7.1  2         arctiq  <all>
consul-2  10.128.0.14:8301    alive   server  1.7.1  2         arctiq  <all>
```

Author Information
------------------

Jacob Mammoliti | jacob.mammoliti@arctiq.ca