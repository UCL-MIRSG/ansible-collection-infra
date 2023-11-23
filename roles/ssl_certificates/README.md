# Ansible Role: mirsg.ssl_certificates

Generate SSL certificates using the [`community.crypto` collection](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html).

## Requirements

### Using the role

If you would like to convert the private key to `pk8` format (`ssl_certificate.use_pk8: true`), you first need to ensure `openssl` is installed before using this role:

```yaml
- name: Install openssl
  ansible.builtin.yum:
    name: openssl
    state: present
  when: ssl_certificate.use_pk8
```

### Testing the role

If you would like to run Ansible Molecule to test this role, the requirements are in [`requirements.txt`](https://github.com/UCL-MIRSG/ansible-role-ssl-certificates/blob/main/requirements.txt).

## Role Variables

There are several variables that **must be set** to use this role.

`ssl_certificate`: dictionary of variables for creating an SSL certificate

### Required variables

The following values **must be included** in the `ssl_certificate` dictionary:

`owner`: name of the user that should own the certificate and associated files

`group`: name of the group that should own the certificate and associated files

`certificate_directory`: directory in which to write the certificate and associated files

`privatekey_filename`: name of the file in which the generated SSL private key will be written

`use_pk8`: boolean; if `true`, will convert the SSL private key to PKCS8 format using the [`community.crypto.openssl_privatekey_convert`](https://docs.ansible.com/ansible/devel/collections/community/crypto/openssl_privatekey_convert_module.html) module

`pk8_filename`: name of the file in which the converted SSL private key will be written. A filename must be provided if `use_pk8` is `true`.

`csr_filename`: name of the file into which the generated OpenSSL certificate signing request will be written

`csr_common_name`: the `commonName` field of the certificate signing request subject

`provider`: name of the provider to use to generate/retrieve the OpenSSL certificate. See the [`community.crypto.x509_certificate`](https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html#parameter-provider) module documentation for options.

### Optional variables

The following are **optional** vaules for the `ssl_certificate` dictionary:

`cache_filename`: location to copy the self-signed certificate to

## Dependencies

- [`community.crypto`](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html)

## Example Playbook

Let's see how to generate self-signed SSL certificates for a PostgreSQL server and client.

First define variables for the server:

> `host_vars/db/vars`

```yaml
# Variables for creating a SSL certificate for a postgresql server
ssl_certificate:
  owner: "root"
  group: "root"
  certificate_directory: "/var/lib/pgsql/certs"
  privatekey_filename: "/var/lib/pgsql/certs/server.key"
  use_pk8: false
  csr_filename: "/var/lib/pgsql/server.csr"
  csr_common_name: "db"
  certificate_filename: "/var/lib/pgsql/server.crt"
  provider: "selfsigned"
  cache_filename: "{{ lookup('env', 'HOME') }}/ansible_persistent_files/pg_certificates/db.postgresql_server.crt"
```

We also need to define variables for the client - here we assume the postgresql client is a tomcat server:

> `host_vars/web/vars`

```yaml
# Variables for creating a SSL certificate for a postgresql client
ssl_certificate:
  owner: "root"
  group: "root"
  certificate_directory: "/usr/share/tomcat/.postgresql"
  privatekey_filename: "/usr/share/tomcat/.postgresql/postgresql.key"
  use_pk8: true
  pk8_filename: "/usr/share/tomcat/.postgresql/postgresql.pk8"
  csr_filename: "/usr/share/tomcat/.postgresql/postgresql.csr"
  csr_common_name: "{{ web_hostname }}"
  certificate_filename: "/usr/share/tomcat/.postgresql/postgresql.crt"
  provider: "selfsigned"
  cache_filename: "{{ lookup('env', 'HOME') }}/ansible_persistent_files/pg_certificates/db.postgresql_client.crt"
```

Then inside our playbook we can use the role:

```yaml
- name: Generate SSL certificates for a postgresql server and client
  hosts: all
  roles:
    - mirsg.ssl_certificates
```

After creating the certificates and (optionally) copying them both to a shared cache, you will need to copy to server certificate to the client and the client certificate to the server.

## License

[BSD 3-Clause License](https://github.com/UCL-MIRSG/ansible-role-ssl-certificates/blob/main/LICENSE).

## Author Information

This role was created by the [Medical Imaging Research Software Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group) at [UCL](https://www.ucl.ac.uk/).
