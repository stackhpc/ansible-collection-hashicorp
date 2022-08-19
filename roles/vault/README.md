This role deploys and initializes Hashicorp Vault with Consul backend

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
  * Optional
    * `consul_container.etc_hosts`: Dict; `{<hostname>:<ip_address>}` to be added to container /etc/host
s (default: Omitted)
    * `vault_extra_volumes`: List of `"<host_location>:<container_mountpoint>"`
    * `vault_tls_key`: Path to TLS key to use by Vault
    * `vault_tls_cert`: Path to TLS cert to use by Vault



Example playbook (used with OpenStack Kayobe)
---------------------------------------------

```
---
- name: Prepare for hashicorp-vault role
  any_errors_fatal: True
  gather_facts: True
  hosts: consul
  tasks:
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
      no_log: True
      become: true

- name: Run hashicorp-vault role
  any_errors_fatal: True
  gather_facts: True
  hosts: consul
  roles:
    - role: stackhpc.hashicorp.vault
      consul_bind_interface: "{{ internal_net_interface }}"
      consul_bind_ip: "{{ internal_net_ips[ansible_hostname] }}"
      vault_bind_address: "{{ external_net_ips[ansible_hostname] }}"
      vault_api_addr: "https://{{ external_net_fqdn }}:8200"
      vault_config_dir: "/opt/kayobe/vault"
```

Example post-config playbook to enable secrets engines:
```
---
- name: Vault post deployment config
  any_errors_fatal: True
  gather_facts: True
  hosts: vault
  tasks:
    - name: Enable vault secrets engines
      hashivault_secret_engine:
        url: "https://sparrow.cf.ac.uk:8200"
        token: "{{ secrets_vault_keys.root_token }}"
        name: pki
        backend: pki
      run_once: True
```

Example vault unseal playbook based on Kayobe's secrets.yml
```
---
- name: Unseal vault
  any_errors_fatal: True
  gather_facts: True
  hosts: vault
  tasks:
    - name: Unseal vault
      hashivault_unseal:
        url: "https://sparrow.cf.ac.uk:8200"
        keys: "{{ item }}"
      run_once: True
      with_items: "{{ secrets_vault_keys.unseal_keys_b64 }}"
      no_log: True
```

NOTE: secrets_external_tls_cert/key are variables in Kayobe's secrets.yml
