This role Generates a Root and Intermediate certificate and generates leaf certificates

Requirements
------------

`ansible-modules-hashivault` Python package installed on the Ansible control host
`hvac` Python package installed on the remote hosts

Note that since version `4.6.4`, `ansible-modules-hashivault` requires
`ansible>4`.

Role variables
--------------

* Vault Create Root
    * `vault_pki_root_create`: Wether to create a RootCA certificate or not (default: `true`)
    * Mandatory if `vault_pki_root_create` equals `true`
        * `vault_pki_root_ca_name`: The name of the RootCA to create (string)
        * `vault_pki_root_ca_common_name`: The common name of the RootCA (default: vault_pki_root_ca_name)
    * `vault_pki_write_root_ca_to_file`: Wether to write the root CA certificate to a file for importing into a systems trust store (default: `false`)

* Vault Create Intermediate
    * `vault_pki_intermediate_create`: Wether to create an intermediate CA or not (default: `true`)
    * `vault_pki_intermediate_import`: Wether to import a pre-existing intermediate pem bundle (default: `false`)
    * `vault_pki_intermediate_export`: Wether to export the generated intermediate pem bundle (default: `false`)

        * Mandatory if `vault_pki_intermediate_create` equals `true`
            * `vault_pki_intermediate_ca_name`: The name of the Intermediate CA to create
            * `vault_pki_intermediate_ca_common_name`:  The common name of the RootCA (default: `vault_pki_intermediate_ca_name`)
        * Mandatory if `vault_pki_intermediate_import`: equals `true`
            * `vault_pki_intermediate_ca_bundle`: Concatenated certificate, intermediate and private key
    * `vault_pki_intermediate_roles`: Certificate Roles to create for the intermediate CA. List of Dicts containing `{name: <role_name>, config: { <pki_option>: <value> ...}`

* Certificate Output
    * `vault_pki_generate_certificates`: whether to generate leaf certificates or not (default: `false`)
    * `vault_pki_write_certificates_host:` The host on which certificates will be written to. (default: "localhost")
    * `vault_pki_certificates_directory`: directory to output certificate files to.
    * `vault_pki_write_certificates`: wether to write generated certificates to a file or not (default: `false`)
    * `vault_pki_certificate_subject`: The certificate subject parameters e.g. `ttl` `ip_sans`. List of Dicts containing `{role: <name of Certificate role>, common_name: <common name of certificate>, extra_params: {ttl: <value>, alt_sans: <value>, ip_sans: <value> }}`