This role deploys and initializes Hashicorp Vault with Consul backend

Requirements
------------

``ansible-modules-hashivault`` PyPI package installed

Role variables
--------------

* Consul
  * Mandatory
    * `consul_bind_interface`: Which interface should be used for Consul
  * Optional
    * `consul_docker_name`: Docker - under which name to run the Consul image (default: "consul")
    * `consul_docker_image`: Docker image for Consul (default: "consul")
    * `consul_docker_tag`: Docker image tag for Consul (default: "latest")
    * `consul_docker_volume`: Docker volume name for Consul data (default: "consul_data")
    * `consul_container.etc_hosts`: Dict of `{<hostname>:<ip_address>}` to be added to container /etc/hosts (default: Omitted)
    * `consul_extra_volumes`: List of `"<host_location>:<container_mountpoint>"`

* Vault
  * Mandatory
    * `vault_cluster_name`: Vault cluster name (e.g. "prod_cluster")
    * `vault_api_addr`: Vault [API addr](https://www.vaultproject.io/docs/configuration#api_addr) - Full URL including protocol and port (e.g. "http://127.0.0.1:8200")
    * `vault_bind_address`: Which IP address should Vault bind to
    * `vault_config_dir`: Directory into which to bind mount Vault configuration
    * `copy_self_signed_ca`: bool; copy a custom CA into the vault container

  * Optional
    * `consul_container.etc_hosts`: Dict; `{<hostname>:<ip_address>}` to be added to container /etc/host
s (default: Omitted)
    * `vault_extra_volumes`: List of `"<host_location>:<container_mountpoint>"`
    * `vault_tls_key`: Path to TLS key to use by Vault
    * `vault_tls_cert`: Path to TLS cert to use by Vault
    * `vault_log_keys`: Whether to log the root token and unseal keys in the Ansible output. Default `false`
    * `vault_set_keys_fact`: Whether to set a `vault_keys` fact containing the root token and unseal keys. Default `false`
    * `vault_write_keys_file`: Whether to write the root token and unseal keys to a file. Default `false`
    * `vault_write_keys_file_host`: Host on which to write root token and unseal keys. Default `localhost`
    * `vault_write_keys_file_path`: Path of file to write root token and unseal keys. Default `vault-keys.json`

Root and unseal keys
--------------------

After Vault has been initialised, a root token and a set of unseal keys are emitted.
It is very important to store these keys safely and securely.
This role provides several mechanisms for extracting the root token and unseal keys:

1. Print to Ansible log output (`vault_log_keys`)
1. Set a `vault_keys` fact (`vault_set_keys_fact`)
1. Write to a file (`vault_write_keys_file`)

In each case, the output will contain the following:

```json
{
  "keys": [
    "...",
    "..."
  ],
  "keys_base64": [
    "...",
    "..."
  ],
  "root_token": "..."
}
```

Example playbook (used with OpenStack Kayobe)
---------------------------------------------

```
---
- name: Prepare for hashicorp-vault role
  any_errors_fatal: true
  gather_facts: true
  hosts: consul
  tasks:
    - name: Copy rootCA
      copy:
        content: "{{ external_ca_cert }}"
        dest: "{{ '/etc/pki/ca-trust/source/anchors/rootCA.crt' if ansible_os_family == 'RedHat' else '/usr/local/share/ca-certificates/rootCA.crt' }}"
        mode: 0600
        no_log: true
      become: true

    - name: update system CA
      become: true
      shell: "{{ 'update-ca-trust' if ansible_os_family == 'RedHat' else 'update-ca-certificates' }}"

    - name: Ensure /opt/kayobe/vault exists
      file:
        path: /opt/kayobe/vault
        state: directory

    - name: Template out tls key and cert
      vars:
        tls_files:
          - content: "{{ secrets_external_tls_cert }}"
            dest: "tls.cert"
          - content: "{{ secrets_external_tls_key }}"
            dest: "tls.key"
      copy:
        content: "{{ item.content }}"
        dest: "/opt/kayobe/vault/{{ item.dest }}"
        owner: 100
        group: 1001
        mode: 0600
      loop: "{{ tls_files }}"
      no_log: true
      become: true

- name: Run hashicorp-vault role
  any_errors_fatal: true
  gather_facts: true
  hosts: consul
  roles:
    - role: stackhpc.hashicorp.vault
      consul_bind_interface: "{{ internal_net_interface }}"
      consul_bind_ip: "{{ internal_net_ips[inventory_hostname] }}"
      vault_bind_address: "{{ external_net_ips[inventory_hostname] }}"
      vault_api_addr: "https://{{ external_net_fqdn }}:8200"
      vault_config_dir: "/opt/kayobe/vault"
```

Example post-config playbook to enable secrets engines:
```
---
- name: Vault post deployment config
  any_errors_fatal: true
  gather_facts: true
  hosts: vault
  tasks:
    - name: Enable vault secrets engines
      hashivault_secret_engine:
        url: "https://sparrow.cf.ac.uk:8200"
        token: "{{ secrets_vault_keys.root_token }}"
        name: pki
        backend: pki
      run_once: true
```

Example vault unseal playbook based on Kayobe's secrets.yml
```
---
- name: Unseal vault
  any_errors_fatal: true
  gather_facts: true
  hosts: vault
  tasks:
    - name: Unseal vault
      hashivault_unseal:
        url: "https://sparrow.cf.ac.uk:8200"
        keys: "{{ item }}"
      run_once: true
      with_items: "{{ secrets_vault_keys.unseal_keys_b64 }}"
      no_log: true
```

NOTE: secrets_external_tls_cert/key and external_ca_cert are variables in Kayobe's secrets.yml
