# autofs_cifs

This role installs and configures `autofs` on the host. Import caveats: the role
can only be used to configure CIFS/Samba mounts and is not tested for CentOS 7.

## Role Variables

All of the following variables are required (no defaults):

- `autofs_mount_point`: Path where the share will be mounted on the host
- `autofs_creds_file`: Path to the file where the credentials for mounting will
  be stored
- `autofs_user`: User that owns the share on the source
- `autofs_password`: Password to use to authenticate as the user that owns the
  source
- `autofs_domain`: Domain where the user who owns the source resides
- `autofs_server`: Address of the server
- `autofs_share`: Name of the share being mounted
- `autofs_owner`: OS user that will own the mounted share
- `autofs_group`: OS group that own the mounted share
- `autofs_map_name`: An identifying name for the share used to name the `autofs`
  configuration file

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: mirsg.infrastructure.autofs_cifs
```
