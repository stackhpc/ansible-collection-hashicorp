This role unseals Hashicorp Vault.

Note that in a Vault cluster, each Vault server must be unsealed individually.

Requirements
------------

`ansible-modules-hashivault` Python package installed on the Ansible control host
`hvac` Python package installed on the remote hosts

Note that since version `4.6.4`, `ansible-modules-hashivault` requires
`ansible>4`.

Role variables
--------------

* `vault_api_addr`: Vault [API addr](https://www.vaultproject.io/docs/configuration#api_addr) - Full URL including protocol and port (e.g. "http://127.0.0.1:8200"). In a Vault cluster, this should point to an individual Vault server, rather than a load balancer.
* `vault_unseal_keys`: List of unseal key shards.

Example playbook
----------------

Example vault unseal playbook:
```
---
- name: Unseal vault
  any_errors_fatal: True
  gather_facts: True
  hosts: vault
  tasks:
    - name: Unseal vault
      import_role:
        name: stackhpc.vault_unseal
      vars:
        vault_api_addr: "https://vault.example.com"
        vault_keys: "{{ vault_keys.keys_base64 }}"
```
