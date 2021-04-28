Ansible Role: Consul
===================
A role to deploy a production grade cluster of [HashiCorp Consul](https://www.consul.io/).

Role Variables
--------------
Ansible variables are listed below, along with default values (see `defaults/main.yml`):

> **NOTE**: The label for servers in the hosts inventory file must be `[consul]` as shown in the example. The role will not properly function if the label name is anything other value.

### `provider`

- Tells Consul where it is installed (useful if using `retry_join` based on Cloud tags)
- Default value: no_cloud

### `consul_cloud_tag`

- Cloud tag Consul will use during `retry_join`
- Default value: consul

### `consul_user`

- OS user
- Default value: consul

### `consul_group`

- OS group
- Default value: consul

### `consul_create_account`

- Whether to create the user and group defined by `consul_user` and `consul_group` or not
- Default value: true

### `consul_home_directory`

- Location of Consul's home directory
- Default value: `/etc/consul.d`

### `consul_data_directory`

- Location of Consul's data directory
- Default value: `/opt/consul`

### `consul_install_directory`

- Location of the Consul binary
- Default value: `/opt/consul/bin`

### `consul_config_file`

- Location of the Consul agent client configuration file
- Default value: `/etc/consul.d/consul.hcl`

### `consul_server_file`

- Location of the Consul agent server configuration file
- Default value: `/etc/consul.d/server.hcl`

### `consul_version`

- Version of Consul to download and install
- Default value: `1.9.3`

### `consul_version`

- Name of the Consul file archive to download
- Default value: `consul_1.9.3_linux_amd64.zip`

### `consul_download`

- Full URL location to download Consul
- Default value: `https://releases.hashicorp.com/consul/1.9.3/consul_1.9.3_linux_amd64.zip`

### `consul_local_binary`

- Whether to use a binary stored locally (this is mutually exclusive with the three above variables)
- Default value: false

### `consul_local_binary_location`

- Location of the local binary
- Default value: `binary/consul`

### `consul_log_level`

- Sets the log level
- Default value: INFO

### `consul_server`

- Whether the Consul agent should be in server mode or not
- Default value: false

### `consul_datacenter`

- Name of the datacenter
- Default value: dc1

### `consul_acl_enabled`

- Whether ACLs are enabled or not
- Default value: false

### `consul_raft_multiplier`

- An integer multiplier used by Consul servers to scale key Raft timing parameters
- Default value: 1

### `consul_ui`

- Whether the UI is enabled or not
- Default value: true

### `consul_client_addr`

- The address to which Consul will bind client interfaces, including the HTTP and DNS servers
- Default value: `0.0.0.0`

### `consul_advertise_addr`

- The advertise address is used to change the address that we advertise to other nodes in the cluster.
- Default value: `ansible_default_ipv4.address`

### `consul_tls_enabled`

- Whether TLS is enabled or not
- Default value: false

### `consul_tls_directory`

- Directory that TLS certificates live in
- Default value: `/etc/consul.d/tls`

### `consul_tls_ca_file`

- Local path to the TLS CA certificate to copy over
- Default value: `tls/consul-agent-ca.pem`

### `consul_tls_cert_file`

- Local path to the TLS signed certificate to copy over
- Default value: `tls/dc1-server-consul-0.pem`

### `consul_tls_key_file`

- Local path to the TLS key to copy over
- Default value: `tls/dc1-server-consul-0-key.pem`

### `consul_tls_verify_incoming`

- Requires that all incoming connections make use of TLS and that the client provides a certificate signed by a Certificate Authority
- Default value: true

### `consul_tls_verify_outgoing`

- Requires that all outgoing connections make use of TLS and that the client provides a certificate signed by a Certificate Authority
- Default value: true

### `consul_tls_verify_server_hostname`

- Consul verifies for all outgoing TLS connections that the TLS certificate presented by the servers matches server.<datacenter>.<domain> hostname
- Default value: true

### `consul_tls_min_version`

- Specifies the minimum supported version of TLS
- Default value: tls12

### `tls_cipher_suites`

- Specifies the list of supported ciphersuites as a comma-separated-list
- Default value: None

### `consul_port_dns`

- Port for DNS (-1 is disabled)
- Default value: `8600`

### `consul_port_http`

- Port for HTTP (-1 is disabled)
- Default value: `8500`

### `consul_port_https`

- Port for HTTPS (-1 is disabled)
- Default value: `-1`

### `consul_port_grpc`

- Port for GRPC (-1 is disabled)
- Default value: `-1`

### `consul_port_serf_lan`

- Port for Serf traffic over LAN 
- Default value: `8301`

### `consul_port_serf_wan`

- Port for Serf traffic over WAN 
- Default value: `8302`

### `consul_port_server`

- Port for RPC
- Default value: `8300`

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
```bash
$ cat <<EOF > inventory
[consul]
consul-0.c.[PROJECT_ID].internal
consul-1.c.[PROJECT_ID].internal
consul-2.c.[PROJECT_ID].internal
EOF
```

Create an Ansible playbook, calling the role:
```bash
$ cat <<EOF > site.yaml
---
- hosts: consul
  become: yes
  roles:
    - role: ansible-role-consul
EOF
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
```bash
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
