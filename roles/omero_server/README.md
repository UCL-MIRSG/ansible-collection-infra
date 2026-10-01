# Installs and configures OMERO.server

This role has been adapted from the
[ome.omero_server](https://github.com/ome/ansible-role-omero-server/tree/master)
role maintained by the OME team. The reasons for maintaining a separate role
here are:

1. The OME role no longer supports EL `8` OS variants
2. There is a
   [bug in the OME role](https://github.com/ome/ansible-role-omero-server/issues/72)
   which stops a database backup working when OMERO.server is upgraded

The tasks from the `ome.omero_common`, `ome.python3_virtualenv`, `ome.ice`,
`ome.deploy_archive` and `ome.basedeps` roles, which this role previously
depended on, have been merged into this role. Those roles are BSD licensed,
copyright the Open Microscopy Environment.

EL `8` and EL `9` are supported.

## Dependencies

A PostgreSQL server installed using `mirsg.infrastructure.postgresql` is
required.

Java must be installed, e.g. using `mirsg.infrastructure.install_java`.

On EL `9` the CodeReady Builder (CRB) repository is enabled to install
`libdb-cxx`, which is needed by the Ice binaries.

See also `mirsg.infrastructure.omero_web` and `mirsg.infrastructure.omero_user`.

## Role Variables

All variables are optional, see defaults/main.yml for the full list

### OMERO.server version

`omero_server_release`: The OMERO release, e.g. 5.6.0. Defaults to `5.6.0`.

`omero_server_dbhost`: Database host

`omero_server_dbuser`: Database user

`omero_server_dbname`: Database name

`omero_server_dbpassword`: Database password

`omero_server_rootpassword`: OMERO root password, defaults to `omero`. This is
only used when initialising a new database.

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

### Ice

`omero_server_ice_archives`: A dictionary, keyed by EL major version, of the Ice
3.6 binary archive to install. Each value has the archive `url`, its `sha256`
checksum and the `root` directory inside the archive.

`omero_server_ice_install_dir`: Where the Ice archive is extracted, and where
the `ice` symlink is created. Defaults to `/opt`.

`omero_server_ice_packages`: A dictionary, keyed by EL major version, of
additional packages needed by the Ice binaries.

`omero_server_system_packages`: System packages installed before OMERO.server,
including Python 3.

`omero_server_virtualenv_command`: Command used to create the OMERO.server
virtualenv. Defaults to `python3 -m venv`.

### OMERO.server systemd configuration

`omero_server_systemd_setup`: Create and start the omero-server systemd service,
default True

`omero_server_systemd_limit_nofile`: Systemd limit for number of open files
(default ignore)

`omero_server_systemd_after`: A list of strings with additional service names to
appear in systemd unit file "After" statements. Default empty/none.

`omero_server_systemd_requires`: A list of strings with additional service names
to appear in systemd unit file "Requires" statements. Default empty/none.

`omero_server_systemd_environment`: Dictionary of additional environment
variables. Python virtualenv

`omero_server_python_addons`: List of additional Python packages to be installed
into virtualenv. Alternatively you can install packages into
/opt/omero/server/venv3 independently from this role. Backups

`omero_server_database_backupdir`: Dump the OMERO database to this directory
before upgrading, default empty (disabled)

### OMERO.mail email configuration

Default values are the same as the
[OMERO.mail defaults](https://docs.openmicroscopy.org/omero/5.6.4/sysadmins/mail.html).

`omero_server_smtp_enabled`: Enable SMTP server. Default `false`

`omero_server_smtp_from`: Send email from this address. Defaults to
`omero@localhost`.

`omero_server_smtp_hostname`: Host to use for sending email. Defaults to
`localhost`

`omero_server_smtp_port`: Send email on this port. Defaults to `25`

`omero_server_smtp_auth`: Require authentication for sending email. Defaults to
`false`

`omero_server_smtp_username`: Authenticate with this user. Defaults to empty
string.

`omero_server_smtp_password`: Authenticate with this password. Default to empty.

`omero_server_smtp_start_tls`: Start TLS connection before logging into the
email server. Defaults to `false`.

### Configuring OMERO.server

This role regenerates the OMERO configuration file using the configuration files
and helper script in `/opt/omero/server/config`. `omero_server_config_set` can
be used for simple configurations, for anything more complex consider creating
one or more configuration files under: `/opt/omero/server/config/` with the
extension .omero.

Manual configuration changes (`omero config ...`) will be lost following a
restart of omero-server with systemd, you can disable this by setting
`omero_server_always_reset_config: false`. Manual configuration changes will
never be copied during an upgrade.

See [ome/design#70](https://github.com/ome/design/issues/70) for a proposal to
add support for a conf.d style directory directly into OMERO.

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
        postgresql_version: "14"
```
