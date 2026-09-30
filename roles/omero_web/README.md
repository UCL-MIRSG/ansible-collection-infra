# Installs and configures OMERO.web

This role has been adapted from the
[ome.omero_web](https://github.com/ome/ansible-role-omero-web) role maintained
by the OME team, and includes the tasks from the `ome.selinux_utils` and
`ome.omero_common` roles it depended on. Those roles are BSD licensed, copyright
the Open Microscopy Environment.

EL `8` and EL `9` are supported.

Unlike the OME role, this role does not install or configure nginx. Use
`mirsg.infrastructure.nginx` with the `omero-web-nginx-conf.j2` template, as in
the [install_omero.yml](../../playbooks/install_omero.yml) playbook. It also
doesn't install redis, add `django-redis` to `omero_web_python_addons` if you
want to use redis for sessions.

## Role Variables

All variables are optional, see [defaults/main.yml](defaults/main.yml) for the
full list.

`omero_web_release`: The OMERO.web version, e.g. `5.28.0`. Defaults to
`present`, which installs the latest version if OMERO.web isn't installed.

`omero_web_basedir`: Base directory for OMERO.web. Defaults to `/opt/omero/web`.

`omero_web_system_user`: OMERO.web system user. Defaults to `omero-web`.

`omero_web_system_uid`: OMERO.web system user ID. Defaults to automatic.

`omero_web_config_set`: A dictionary of config-key: value used to configure
OMERO.web. Values can be strings or objects (lists, dictionaries), which are
converted to quoted JSON.

`omero_web_always_reset_config`: Drop manual configuration changes when
omero-web is restarted. Defaults to `true`.

`omero_web_python_addons`: Additional Python packages to install into the
virtualenv.

### OMERO.web apps

`omero_web_apps_packages`: Python packages for OMERO.web apps.

`omero_web_apps_names`: Names of apps to add to `omero.web.apps`.

`omero_web_apps_top_links`: A list of dictionaries with keys `label`, `link` and
optionally `attrs`, added to `omero.web.ui.top_links`.

`omero_web_apps_ui_metadata_panes`: Items to add to
`omero.web.ui.metadata_panes`.

`omero_web_apps_config_append`: A dictionary of config-key: list of values to
append.

`omero_web_apps_config_set`: A dictionary of config-key: value to set.

### systemd

`omero_web_systemd_setup`: Create the omero-web systemd service. Defaults to
`true`.

`omero_web_systemd_start`: Start the omero-web service. Defaults to `true`.

`omero_web_systemd_limit_nofile`: systemd limit on the number of open files.

`omero_web_systemd_after` / `omero_web_systemd_requires`: Additional services
for the `After` and `Requires` statements in the unit file.

### SELinux

If SELinux is enabled on the host, the role sets the booleans and port needed
for nginx to proxy OMERO.web, and installs a policy module allowing nginx to
serve the OMERO.web static files.

`omero_web_selinux_manage`: Configure SELinux. Defaults to `true`.

`omero_web_selinux_port`: The OMERO.web port nginx connects to. Defaults to
`4080`.

`omero_web_selinux_policy_dir`: Where the SELinux policy module is built.
Defaults to `/usr/local/share/selinux/omero-web`.

## Example Playbook

```yaml
- hosts: omero
  roles:
    - role: mirsg.infrastructure.omero_web
      vars:
        omero_web_config_set:
          omero.web.public.enabled: true
```
