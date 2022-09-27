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
* `vault_unseal_authtype`: authentication type
* `vault_unseal_aws_header`: X-Vault-AWS-IAM-Server-ID Header value to prevent replay attacks
* `vault_unseal_ca_cert`: Path to a PEM-encoded CA cert file to use to verify the Vault server TLS certificate
* `vault_unseal_ca_path`: Path to a directory of PEM-encoded CA cert files to verify the Vault server TLS certificate. If `vault_unseal_ca_cert` is specified, its value will take precedence
* `vault_unseal_client_cert`: Path to a PEM-encoded client certificate for TLS authentication to the Vault server
* `vault_unseal_client_key`: Path to an unencrypted PEM-encoded private key matching the client certificate
* `vault_unseal_keys`: List of unseal key shards.
* `vault_unseal_login_mount_point`: Authentication mount point
* `vault_unseal_namespace`: Namespace for Vault
* `vault_unseal_password`: Password for Vault
* `vault_unseal_token`: Token for Vault
* `vault_unseal_username`: Username to login to Vault
* `vault_unseal_verify`: If set, do not verify presented TLS certificate before communicating with Vault server.

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
