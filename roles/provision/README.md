# Ansible Role: mirsg.infrastructure.provision

This role sets up for a specific distribution for CentOS (mirrorlist and locale)
or Rocky8 (disable postgres), upgrades all packages and ensures epel is
installed.

## Role Variables

The following variables can be set for provisioning CentOS 7:

| Name                                    | Description                                                                                                       |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `provision_yum_repo_base_baseurl`       | baseurl for YUM base repo. Defaults to `http://vault.centos.org/centos/$releasever/os/$basearch/`                 |
| `provision_yum_repo_updates_baseurl`    | baseurl for YUM updates repo updates. Defaults to `http://vault.centos.org/centos/$releasever/updates/$basearch/` |
| `provision_yum_repo_extras_baseurl`     | baseurl for YUM extra repo extras. Defaults to `http://vault.centos.org/centos/$releasever/extras/$basearch/`     |
| `provision_yum_repo_centosplus_baseurl` | baseurl for YUM centosplus repo. Defaults to `http://vault.centos.org/centos/$releasever/centosplus/$basearch/`   |

The following variables can be set for provisioning Rocky 8+:

| Name                                  | Description                         |
| ------------------------------------- | ----------------------------------- |
| `postgresql_rpm_gpg_key_pgdg_x86_64`  | the postgresql key for Intel chips. |
| `postgresql_rpm_gpg_key_pgdg_aarch64` | the postgresql key for ARM chips    |

The following variables can be set for either CentOS 7 or Rocky 8+:

| Name                     | Description                                                          |
| ------------------------ | -------------------------------------------------------------------- |
| `server_locale`          | sets the user's language, region, etc. This is set to "en_GB.UTF-8"  |
| `external_storage_drive` | path to mounted storage (if using it). By default this is undefined. |

## Dependencies

There are no Ansible-Galaxy dependencies for this role.

## Example Playbook

This role will perform provision tasks on a managed host. To use this role, add
it to the list of roles in a play:

```yaml
- name: Provision
  hosts: all
  roles:
    - mirsg.infrastructure.provision
```
