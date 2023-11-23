# Ansible Role: mirsg.postgresql

This role will install and configure a PostgreSQL database on a server.

This role is tested on the following operating systems:

- CentOS 7
- Rocky Linux 8

## Requirements

If you would like to run Ansible Molecule to test this role, the requirements are in [`requirements.txt`](https://github.com/UCL-MIRSG/ansible-role-postgresql/blob/main/requirements.txt).

## Role Variables

There are several **optional** variables for the PostgreSQL server. See the [defaults here](defaults/main.yml).

There are also several **required** variables you will need to set before using this role.

### Required variables for the PostgreSQL server

See [this example](molecule/resources/inventory/host_vars/db/vars) `host_vars` file.

### Required variables for the PostgreSQL client

See [this example](molecule/resources/inventory/host_vars/web/vars) `host_vars` file.

### Required varaibles for both the server and client

`postgresql_rpm_gpg_key_pgdg`: URL from which to download the RPM GPP key; not needed for CentOS 7

`postgresql_use_ssl:` boolean; whether to use SSL

## Dependencies

If `postgresql_use_ssl` is set to `true`, `mirsg.ssl_certificates` must be installed and the `ssl_certificate`
variable defined for both the client and server.

## Example Playbook

To use this role you must first stop any running PostgreSQL service and, if necessary, create a SSL certificate
for the client:

```yaml
- name: Disable default postgresl module and install rpm key
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Disable default postgresl module and install rpm key on RedHat 8+
      when: ansible_facts['os_family'] == "RedHat" and ansible_facts['distribution_major_version'] is version('7', '>')
      block:
        - name: Disable default Postgres module # noqa command-instead-of-module
          ansible.builtin.command: yum module disable -y postgresql
          register: disable_postgresql_module
          changed_when:
            - "'Disabling modules:' in disable_postgresql_module.stdout"

        - name: Import postgresql rpm key
          ansible.builtin.rpm_key:
            state: present
            key: "{{ postgresql_rpm_gpg_key_pgdg }}"

- name: Create client SSL certificate
  hosts: web
  become: true
  gather_facts: true
  tasks:
    - name: Install openssl
      ansible.builtin.yum:
        name: openssl
        state: present

    - name: Create self-signed SSL certificate for the client
      ansible.builtin.include_role:
        name: "mirsg.ssl_certificates"
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
        name: "mirsg.postgresql"
```

## License

[BSD 3-Clause License](https://github.com/UCL-MIRSG/ansible-role-postgresql/blob/main/LICENSE).

## Author Information

This role was created by the [Medical Imaging Research Software Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group) at [UCL](https://www.ucl.ac.uk/).
