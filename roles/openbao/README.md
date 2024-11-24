This role deploys and initializes OpenBao with a Raft backend.

Requirements
------------

`ansible-modules-hashivault` Python package installed on the Ansible control host
`hvac` Python package installed on the remote hosts

Note that since version `4.6.4`, `ansible-modules-hashivault` requires
`ansible>4`.

Role variables
--------------

* Common variables
  * Optional
    * `openbao_registry_url`: Address of the Docker registry used to authenticate (default: "")
    * `openbao_registry_username`: Username used to authenticate with the Docker registry (default: "")
    * `openbao_registry_password`: Password used to authenticate with the Docker registry (default: "")

* OpenBao
  * Mandatory
    * `openbao_cluster_name`: OpenBao cluster name (e.g. "prod_cluster")
    * `openbao_config_dir`: Directory into which to bind mount OpenBao configuration
  * Optional
    * `openbao_bind_address`: Which IP address should OpenBao bind to (default: "127.0.0.1")
    * `openbao_api_addr`: OpenBao [API addr](https://openbao.org/docs/configuration/#high-availability-parameters) - Full URL including protocol and port (default: "http://127.0.0.1:8200")
    * `openbao_init_addr`: OpenBao init addr (used only for initialisation purposes) - full URL including protocol and port (default: "http://127.0.0.1:8200")
    * `openbao_docker_name`: Docker - under which name to run the OpenBao image (default: "bao")
    * `openbao_docker_image`: Docker image for OpenBao (default: "openbao/openbao")
    * `openbao_docker_tag`: Docker image tag for OpenBao (default: "latest")
    * `openbao_extra_volumes`: List of `"<host_location>:<container_mountpoint>"`
    * `openbao_ca_cert`: Path to CA certificate used to verify OpenBao server TLS cert
    * `openbao_tls_key`: Path to TLS key to use by OpenBao
    * `openbao_tls_cert`: Path to TLS cert to use by OpenBao
    * `openbao_log_keys`: Whether to log the root token and unseal keys in the Ansible output. Default `false`
    * `openbao_set_keys_fact`: Whether to set a `openbao_keys` fact containing the root token and unseal keys. Default `false`
    * `openbao_write_keys_file`: Whether to write the root token and unseal keys to a file. Default `false`
    * `openbao_write_keys_file_host`: Host on which to write root token and unseal keys. Default `localhost`
    * `openbao_write_keys_file_path`: Path of file to write root token and unseal keys. Default `bao-keys.json`

Root and unseal keys
--------------------

After OpenBao has been initialised, a root token and a set of unseal keys are emitted.
It is very important to store these keys safely and securely.
This role provides several mechanisms for extracting the root token and unseal keys:

1. Print to Ansible log output (`openbao_log_keys`)
1. Set a `openbao_keys` fact (`openbao_set_keys_fact`)
1. Write to a file (`openbao_write_keys_file`)

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
- name: Prepare for OpenBao role
  any_errors_fatal: True
  gather_facts: True
  hosts: consul
  tasks:
    - name: Ensure /opt/kayobe/bao exists
      file:
        path: /opt/kayobe/bao
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
        dest: "/opt/kayobe/bao/{{ item.dest }}"
        owner: 100
        group: 1001
        mode: 0600
      loop: "{{ tls_files }}"
      no_log: True
      become: true

- name: Run OpenBao role
  any_errors_fatal: True
  gather_facts: True
  hosts: consul
  roles:
    - role: stackhpc.hashicorp.openbao
      openbao_bind_address: "{{ external_net_ips[inventory_hostname] }}"
      openbao_api_addr: "https://{{ external_net_fqdn }}:8200"
      openbao_config_dir: "/opt/kayobe/bao"
```

Example post-config playbook to enable secrets engines:
```
---
- name: OpenBao post deployment config
  any_errors_fatal: True
  gather_facts: True
  hosts: bao
  tasks:
    - name: Enable bao secrets engines
      hashivault_secret_engine:
        url: "https://vault.example.com:8200"
        token: "{{ secrets_openbao_keys.root_token }}"
        name: pki
        backend: pki
      run_once: True
```

NOTE: secrets_external_tls_cert/key are variables in Kayobe's secrets.yml
