Vault to OpenBao migration playbook
-----------------------------------
`vault_bao_migration.yml` provides full play of Vault to OpenBao migration route for both single node and HA clustered Vault.

Playbook Variables
------------------

* Mandatory
    * `vault_hosts_group`: Name of ansible inventory group that runs Vault

`vault_bao_migration` Role Variables
------------------------------------

This playbook requires variables for the [`vault_bao_migration`](../roles/vault_bao_migration/README.md) role to be set. Please refer to the README of the role for more information about its variables.

Playbook Tags
-------------

You can run/skip part of the tasks by using following tags

* ``raft`` includes tasks for
  - Taking Consul snapshot
  - Migrating storage backend to Raft
  - Stopping Consul container
* ``migration`` includes tasks for
  - Taking Raft snapshot (Single node only)
  - Migrating Vault to OpenBao

Prerequiste tasks are always run

Example usage
-------------
```
- name: Migrate Vault to Openbao
  import_playbook: stackhpc.hashicorp.vault_bao_migration
  vars:
    vault_hosts_group: vault
    # These are vault_bao_migration role variables
    migration_common_root_token: "{{ secret_store_keys.root_token }}"
    migration_common_unseal_keys: "{{ secret_store_keys.keys_base64 }}"
    migration_common_cluster_name: prod
    migration_vault_config_dir: /opt/kayobe/vault
    migration_openbao_config_dir: /opt/kayobe/openbao

```
