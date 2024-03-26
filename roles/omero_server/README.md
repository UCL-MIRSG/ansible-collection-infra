# Installs and configures OMERO.server

This role has been adapted from the
[ome.omero_server](https://github.com/ome/ansible-role-omero-server/tree/master)
role maintained by the OME team. The reasons for maintaining a separate role
here are:

1. The OME role no longer supports EL `8` OS variants
2. There is a [bug in the OME
   role](https://github.com/ome/ansible-role-omero-server/issues/72) which stops
   a database backup working when OMERO.server is upgraded

If running EL `9` and you set `omero_server_release` to `latest` it is
recommended that you use the `ome.omero_server` role in place of this one.

## Dependencies

A PostgreSQL server installed using `mirsg.infrastructure.postgresql` is
required.

This role is also dependent on a number of other OME roles. Due to the lack of
support for EL `8` OS variants in newer versions of those roles, the versions of
these dependencies are pinned here (see the collection
[requirements.yml](../../meta/requirements.yml) file):

| Role                   | Version |
| ---------------------- | ------- |
| ome.omero_web          | 4.0.1   |
| ome.omero_user         | 0.3.1   |
| ome.selinux_utils      | 2.0.2   |
| ome.omero_common       | 0.4.0   |
| ome.python3_virtualenv | 0.2.0   |
| ome.ice                | 4.3.0   |

Note that these roles themselves have dependencies on other OME roles.

## Role Variables

All variables are optional, see defaults/main.yml for the full list

### OMERO.server version

`omero_server_release`: The OMERO release, e.g. 5.6.0. Defaults to `5.6.0`.

`omero_server_dbhost`: Database host

`omero_server_dbuser`: Database user

`omero_server_dbname`: Database name

`omero_server_dbpassword`: Database password

`omero_server_rootpassword`: OMERO root password, defaults to `omero`.
This is only used when initialising a new database.

### OMERO.server configuration

`omero_server_config_set`: A dictionary of config-key: value which will be used
for the initial OMERO.server configuration, default empty. value can be a
string, or an object (list, dictionary) that will be automatically converted to
quoted JSON. Note configuration can also be done pre/post installation using the
server/config conf.d style directory. OMERO system user, group, permissions, and
data directory. You may need to change these for in-place imports.

`omero_server_system_user`: OMERO.server system user, default omero-server.
`omero_server_system_user_manage`: Create or update the OMERO.server system user
if necessary, default True. `omero_server_system_uid`: OMERO system user ID
(default automatic) `omero_server_system_umask`: OMERO system user umask, may
need to be changed for in-place imports `omero_server_system_managedrepo_group`:
OMERO system group for the ManagedRepository `omero_server_datadir_mode`:
Permissions for OMERO data directories apart from ManagedRepository
`omero_server_datadir_managedrepo_mode`: Permissions for OMERO ManagedRepository
`omero_server_datadir`: OMERO data directory, default /OMERO
`omero_server_datadir_managedrepo`: OMERO ManagedRepository directory
`omero_server_selfsigned_certificates`: Generate self-signed certificates
instead of using anonymous ciphers, default True, use this if your system does
not support insecure ciphers

### OMERO.server systemd configuration

`omero_server_systemd_setup`: Create and start the omero-server systemd service,
default True

`omero_server_systemd_limit_nofile`: Systemd limit for number of
open files (default ignore)

`omero_server_systemd_after`: A list of strings with
additional service names to appear in systemd unit file "After" statements.
Default empty/none.

`omero_server_systemd_requires`: A list of strings with
additional service names to appear in systemd unit file "Requires" statements.
Default empty/none.

`omero_server_systemd_environment`: Dictionary of additional
environment variables. Python virtualenv

`omero_server_python_addons`: List of additional Python packages to be installed
into virtualenv. Alternatively you can install packages into
/opt/omero/server/venv3 independently from this role. Backups

`omero_server_database_backupdir`: Dump the OMERO database to this directory
before upgrading, default empty (disabled)

### Configuring OMERO.server

This role regenerates the OMERO configuration file using the configuration files
and helper script in `/opt/omero/server/config`. `omero_server_config_set` can be
used for simple configurations, for anything more complex consider creating one
or more configuration files under: `/opt/omero/server/config/` with the
extension .omero.

Manual configuration changes (omero config ...) will be lost following a restart
of omero-server with systemd, you can disable this by setting
omero_server_always_reset_config: false. Manual configuration changes will never
be copied during an upgrade.

See ome/design#70 for a proposal to add support for a conf.d style directory
directly into OMERO.

### Example Playbook

```yaml
# Install or upgrade to a particular version, with an external database
- hosts: localhost
  roles:
    - role: mirsg.infrastructure.omero_server
      vars:
        omero_server_release: "5.6.0"
        omero_server_dbhost: postgres.example.org
        omero_server_dbuser: db_user
        omero_server_dbname: db_name
        omero_server_dbpassword: db_password
        # Version required for the psql client
        postgresql_version: "13"
```
