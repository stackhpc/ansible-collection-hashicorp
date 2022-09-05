# Example playbook

* vault backend and backend HA
  * Mandatory
    * `vault_root_ca_name`: The common name for the RootCA
    * `vault_intermediate_ca_name`: The common name for the intermediateCA

This playbook will create vault self-signed certificates for vault running in HA mode on 3 nodes.

It uses a loadbalancer IP address. it can also use an FQDN by replacing ip_sans with alt_names on the generate certificates task.

```
- name: Generate vault self-signed certificates
  any_errors_fatal: true
  gather_facts: true
  hosts: consul
  vars:
    consul_bind_interface: "{{ internal_net_interface }}"
    consul_bind_ip: "{{ internal_net_ips[ansible_hostname] }}"
    consul_vip_address: "{{ internal_net_vip_address }}"
    vault_bind_address: "{{ external_net_ips[ansible_hostname] }}"
    vault_api_addr: "https://{{ external_net_fqdn }}:8200"
    vault_config_dir: "/opt/kayobe/vault"
  tasks:
    - name: Generate the certificates
      command: >-
        docker exec -e 'VAULT_ADDR={{ vault_api_addr }}' {{ vault_docker_name }}
        vault write {{ vault_intermediate_ca_name }}/issue/ServerCert common_name='vault'
        ip_sans='{{ kolla_internal_vip_address }}' -format=json
      changed_when: false
      register: certs_and_keys_output

    - name: "Set vault cert and keys"
      set_fact:
        vault_cert_and_key: "{{ certs_and_keys_output.stdout | from_json }}"

    - name: "Create vault cert for export"
      copy:
        dest: /tmp/vault.crt
        content: |
            {{ vault_cert_and_key.data.certificate }}
            {{ vault_cert_and_key.data.issuing_ca }}
        mode: '0400'
      delegate_to: localhost

    - name: "Create vault private key for export"
      copy:
        dest: /tmp/vault.key
        content: "{{ vault_cert_and_key.data.private_key }}"
        mode: '0400'
      delegate_to: localhost
```
