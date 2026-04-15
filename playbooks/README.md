Vault to OpenBao migration playbook
-----------------------------------
`vault_bao_migration.yml` provides full play of Vault to OpenBao migration route for both single node and HA clustered Vault.

Requirements
-------------

This playbook uses set of tasks from the role ``vault_bao_migration``.

This playbook assumes that the CA of Vault/OpenBao's TLS is already trusted on the hosts that runs it.

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

Prerequisite tasks are always run

Raft Migration
--------------

Before OpenBao migration, Vault's storage backend needs to be migrated to Raft.

> IMPORTANT: If some Vault nodes fail to migrate to Raft backend in HA setting,
> it can be easier to just continuing with OpenBao migration with the remianing
> cluster and replicate missing nodes later. However, since Raft needs
> to maintain quorum, it is important to check if it's safe to push
> forward. You can check the failure tolerance of the cluster by querying
> Raft state with [HTTP API request](https://developer.hashicorp.com/vault/api-docs/system/storage/raftautopilot#get-cluster-state)
> or [Vault CLI](https://developer.hashicorp.com/vault/docs/commands/operator/raft#autopilot-state)
> inside the Vault container. If it's zero, OpenBao migration cannot go
> ahead and Raft cluster needs to be fixed until the tolerance becomes
> greater than zero.

Example Usage
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
