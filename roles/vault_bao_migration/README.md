This role provides set of tasks that can be used when migrating Hashicorp Vault to OpenBao.
A playbook that provides ready-to-use migration path can be found at `playbooks/vault_bao_migration.yml`
or `stackhpc.hashicorp.vault_bao_migration` with import_playbook task.

Requirements
------------

`ansible-modules-hashivault` Python package installed on the Ansible control host
`hvac` Python package installed on the remote hosts

Note that since version `4.6.4`, `ansible-modules-hashivault` requires
`ansible>4`.

This role assumes that the CA of Vault/OpenBao's TLS is already trusted on the hosts that runs it.

Migration Constraints
---------------------

* Vault version 1.14.1 (OSS)
* OpenBao version 2.2.0
* Shamir unseal (no auto-unseal)

These contraints are based on OpenBao's guide [In-Place Migration from Vault CE](https://openbao.org/docs/guides/migration/)

Role variables
--------------

* Common variables
  * Mandatory
    * `migration_common_root_token`: Root token string of current Vault cluster
    * `migration_common_unseal_keys`: List of unseal key shards.
  * Optional
    * `migration_common_cluster_name`: Name of the current Vault cluster e.g. "prod_cluater" (default: "")
    * `migration_common_bind_addr`: Which IP address should OpenBao bind to (default: "127.0.0.1")
    * `migration_common_api_addr`: OpenBao [API addr](https://openbao.org/docs/configuration/#high-availability-parameters) - Full URL including protocol and port (default: "http://127.0.0.1:8200")
    * `migration_common_init_addr`: OpenBao init addr (used only for initialisation purposes) - full URL including protocol and port (default: "http://127.0.0.1:8200")
    * `migration_common_tls_key`: Path to TLS key to use by Vault and OpenBao
    * `migration_common_tls_cert`: Path to TLS cert to use by Vault and OpenBao
    * `migration_common_tls_ca`: Path to TLS CA certificate that can be used by peers to validate the leaders TLS

* Hashicorp Vault
  * Mandatory
    * `migration_vault_config_dir`: Directory where current Vault configuration is mounted
  * Optional
    * `migration_consul_docker_name`: The name of Consul container that is running (default: "consul")
    * `migration_consul_bind_port`: The port that is currently used by Consul (default: "8500")
    * `migration_vault_docker_name`: The name of Vault container that is running (default: "vault")
    * `migration_vault_docker_image`: Docker image for Vault (default: "hashicorp/vault")
    * `migration_vault_docker_tag`: Docker image tag for Vault (default: "1.14.1)
    * `migration_vault_extra_volumes`: List of `"<host_location>:<container_mountpoint>"` that were configured to current Vault cluster
    * `migration_vault_container_options.etc_hosts`: Dict; `{<hostname>:<ip_address>}` to be added to container /etc/host

* OpenBao
  * Mandatory
    * `migration_openbao_config_dir`: Directory into which to bind mount OpenBao configuration
  * Optional
    * `migration_openbao_registry_url`: Address of the Docker registry used to authenticate (default: "")
    * `migration_openbao_registry_username`: Username used to authenticate with the Docker registry (default: "")
    * `migration_openbao_registry_password`: Password used to authenticate with the Docker registry (default: "")
    * `migration_openbao_docker_name`: Docker - under which name to run the OpenBao image (default: "bao")
    * `migration_openbao_docker_image`: Docker image for OpenBao (default: "openbao/openbao")
    * `migration_openbao_docker_tag`: Docker image tag for OpenBao (default: "2.2.0")
    * `migration_openbao_write_keys_file_host`: Host on which to write root token and unseal keys. (default: `localhost`)
    * `migration_openbao_write_keys_file_path`: Path of file to write root token and unseal keys. (default: `bao-keys.json`)
    * `migration_openbao_enable_ui`: Whether to enable user interface that could be accessed from the `openbao_api_addr`. Default `false`

Example use with `vault_bao_migration` playbook
-----------------------------------------------
```
- name: Migrate Vault to Openbao
  import_playbook: stackhpc.hashicorp.vault_bao_migration
  vars:
    vault_hosts_group: vault
    migration_common_root_token: "{{ secret_store_keys.root_token }}"
    migration_common_unseal_keys: "{{ secret_store_keys.keys_base64 }}"
    migration_common_cluster_name: prod
    migration_vault_config_dir: /opt/kayobe/vault
    migration_openbao_config_dir: /opt/kayobe/openbao

```
