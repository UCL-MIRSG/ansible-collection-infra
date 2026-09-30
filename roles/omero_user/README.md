# Creates OMERO users and groups

This role has been adapted from the
[ome.omero_user](https://github.com/ome/ansible-role-omero-user) role maintained
by the OME team. That role is BSD licensed, copyright the Open Microscopy
Environment.

It must run on the OMERO.server host after OMERO.server has started, e.g. after
`mirsg.infrastructure.omero_server`. Resetting passwords uses `psql`, which is
installed by `mirsg.infrastructure.omero_server`.

## Role Variables

See [defaults/main.yml](defaults/main.yml) for the defaults.

`omero_user_system`: The OMERO.server system user, used to run the `omero` CLI.

`omero_user_bin_omero`: Path to the `omero` CLI.

`omero_user_admin_user` / `omero_user_admin_pass`: OMERO admin credentials used
to create users and groups.

`omero_group_create`: A list of groups to create, each a dictionary with keys
`name` and `type` (e.g. `read-only`). Existing groups are not changed.

`omero_user_create`: A list of users to create, each a dictionary with keys
`login`, `firstname`, `lastname`, `password` and `groups` (arguments passed to
`omero user add`, e.g. `--group-name system`). Existing users are not changed,
unless `force: true` is set, in which case their password is reset.

`omero_user_reset_root_password`: Reset the OMERO root password to this value.
Defaults to empty (disabled).

`omero_user_dbhost` / `omero_user_dbuser` / `omero_user_dbname` /
`omero_user_dbpassword`: Database connection used to reset passwords.

## Example Playbook

```yaml
- hosts: omero
  roles:
    - role: mirsg.infrastructure.omero_user
      vars:
        omero_user_system: omero-server
        omero_user_admin_pass: "{{ vault_omero_rootpassword }}"
        omero_group_create:
          - name: public
            type: read-only
        omero_user_create:
          - login: public_user
            firstname: public
            lastname: user
            password: "{{ vault_public_user_password }}"
            groups: --group-name public
```
