# Ansible Role: mirsg.infrastructure.postgresql

This role will install and configure a PostgreSQL database on a server.

This role is tested on the following operating systems:

- CentOS 7
- Rocky Linux 9

## Role Variables

There are several **optional** variables for the PostgreSQL server. See the
[defaults here](defaults/main.yml).

There are also several **required** variables you will need to set before using
this role.

### Variables required by both the server and client

| Name                                 | Description                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------- |
| `postgresql_rpm_gpg_key_pgdg_x86_64` | URL from which to download the RPM GPP key for Intel chips; not needed for CentOS 7 |
| `postgresql_rpm_gpg_key_pgdg_x86_64` | URL from which to download the RPM GPP key for ARM chips; not needed for CentOS 7   |
| `postgresql_use_ssl`                 | Whether to use SSL                                                                  |

### Required variables for the PostgreSQL server

| Name                                                | Description                                                                                                  |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `postgresql_database.database_name`                 | Name of the database to create                                                                               |
| `postgresql_database.user_name`                     | User for the database                                                                                        |
| `postgresql_database.user_password`                 | Password for the database (this should be stored in an Ansible Vault)                                        |
| `postgresql_connection.host`                        | Hostname of the server                                                                                       |
| `postgresql_connection.port`                        | Port that PostgreSQL is exposed on (should be 5432)                                                          |
| `postgresql_connection.client_ip`                   | IP address from which to allow incoming connections                                                          |
| `postgresql_connection.client_certificate_filename` | Where to copy the client certificate to on the server; only required if using `postgresql_use_ssl` is `true` |
| `postgresql_connection.listen_address`              | IP address(es) the server should listen on                                                                   |
| `postgresql_connection.subnet_mask`                 | Subnet mask to apply to `postgresql_connection.client_ip` for incoming connections                           |

Note, if `postgresql_use_ssl` is set to `true`, you will also need to define a
`postgresql_ssl_certificate` variable. for generating the server certificate.
Generation of new certificates can be disabled by setting
`postgresql_generate_certs` to `false` (defaults to `true`).

See the
[`mirsg.infrastructure.ssl_certificates` README](../ssl_certificates/README.md)
for a description of how to define this variable.

### Required variables for the PostgreSQL client

| Name                                                          | Description                                           |
| ------------------------------------------------------------- | ----------------------------------------------------- |
| `postgresql_client_configuration.server_certificate_filename` | Where to copy the server certificate to on the client |

Note, if `postgresql_use_ssl` is set to `true`, you can use
[`mirsg.infrastructure.ssl_certificates`](../ssl_certificates/README.md) to
generate an SSL certificate.

## Example Playbook

To use this role with a dual-server setup (a dartase `db` and a web server
`web`) you must first:

- stop any running PostgreSQL service
- create a SSL certificate for the client if using SSL:

```yaml
- name: Disable default postgresl module and install rpm key
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Disable default postgresl module and install rpm key on RedHat 8+
      when:
        ansible_facts['os_family'] == "RedHat" and
        ansible_facts['distribution_major_version'] is version('7', '>')
      block:
        - name: Disable default Postgres module # noqa command-instead-of-module
          ansible.builtin.command: yum module disable -y postgresql
          register: disable_postgresql_module
          changed_when:
            - "'Disabling modules:' in disable_postgresql_module.stdout"

        - name: Import postgresql rpm key
          ansible.builtin.rpm_key:
            state: present
            key: "{{ postgresql_rpm_gpg_key_pgdg_x86_64 }}"

- name: Create client SSL certificate
  hosts: web
  become: true
  gather_facts: true
  tasks:
    - name: Install openssl
      ansible.builtin.dnf:
        name: openssl
        state: present

    - name: Create self-signed SSL certificate for the client
      ansible.builtin.include_role:
        name: "mirsg.infrastructure.ssl_certificates"
      vars:
        ssl_certificate: "{{ postgresql_client_ssl_certificate }}"
```

You can then use this role to install and configure PostgreSQL on a server:

```yaml
- name: Create and setup a postgresql database
  hosts: db
  become: true
  gather_facts: true
  tasks:
    - name: Create and setup a postgresql database
      ansible.builtin.include_role:
        name: "mirsg.infrastructure.postgresql"
```
