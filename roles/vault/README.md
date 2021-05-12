This role deploys and initializes Hashicorp Vault with Consul backend

Role variables
--------------

* Consul
  * Mandatory
    * `consul_bind_interface`: Which interface should be used for Consul
    * `consul_vip_address`: Under which IP address consul should be available (this role does not deploy keepalived)
  * Optional
    * `consul_docker_name`: Docker - under which name to run the Consul image (default: "consul")
    * `consul_docker_image`: Docker image for Consul (default: "consul")
    * `consul_docker_tag`: Docker image tag for Consul (default: "latest")
    * `consul_docker_volume`: Docker volume name for Consul data (default: "consul_data")
* Vault
  * Mandatory
    * `vault_cluster_name`: Vault cluster name (e.g. "prod_cluster")
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
        state: present
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
      consul_bind_ip: "{{ internal_net_ips[ansible_hostnanme] }}"
      consul_vip_address: "{{ internal_net_vip_address }}"
      vault_bind_address: "{{ external_net_ips[ansible_hostname] }}"
      vault_vip_url: "{{ external_net_fqdn }}"
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
