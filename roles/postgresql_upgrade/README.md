# Ansible Role: mirsg.postgresql_upgrade

This role upgrades the version of an existing Postgresql cluster. To use this
role the existing Postgresql cluster must have been installed using the
[mirsg.infrastructure.postgresql](../postgresql/README.md) role.

## Role Variables

| Name                                  | Description                                                                                                                                        | Default                                                          |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `postgreql_upgrade_backup_script`     | The path to the backup script generated after installation of Postgresql using the [mirsg.infrastructure.postgresql](../postgresql/README.md) role | `/var/lib/pgsql/run_db_backup.sh`                                |
| `postgresql_upgrade_postgresql_owner` | The Postgresql OS user                                                                                                                             | `postgres`                                                       |
| `postgresql_upgrade_postgresql_group` | The default OS group for the Postgresql user                                                                                                       | `postgres`                                                       |
| `postgreql_upgrade_current_version`   | The Postgresql version of the existing installation                                                                                                |                                                                  |
| `postgreql_upgrade_new_version`       | The desired Postgresql version                                                                                                                     |                                                                  |
| `postgresql_upgrade_data_dir`         | The data folder of the new installation (used to check whether an upgrade has already occurred)                                                    |                                                                  |
| `postgresql_upgrade_scripts_dir`      | The path to the generated Postgresql upgrade scripts                                                                                               | `/var/lib/pgsql/{{ postgreql_upgrade_current_version }}/upgrade` |

## Dependencies

There are no Ansible-Galaxy dependencies for this role.

## Example Playbook

This role will perform postgresql_upgrade tasks on a managed host. Note that to
use this role you will need to make sure that any applications that use the
associated Postgresql database are stopped before running the role. For
convenience a [playbook is provided in this
collection](../../playbooks/upgrade_postgresql.yml). To use this role, add it to
the list of roles in a play:

```yaml
- name: Upgrade Postgresql from version 12 to 14
  hosts: all
  vars:
    postgreql_upgrade_current_version: 12
    postgreql_upgrade_new_version: 14
    postgresql_upgrade_data_dir: "/var/lib/pgsql/{{ postgreql_upgrade_new_version }}/data"
  roles:
    - mirsg.postgresql_upgrade
```
