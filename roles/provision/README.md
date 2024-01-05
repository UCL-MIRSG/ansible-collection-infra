# Ansible Role: mirsg.provision

This role sets up for a specific distribution for CentOS (mirrorlist and locale) or Rocky8 (disable postgres), upgrades all packages and ensures epel is installed.

## Requirements

If you would like to run Ansible Molecule to test this role, the requirements are in [`requirements.txt`](https://github.com/UCL-MIRSG/ansible-role-install-python/blob/main/requirements.txt).

## Role Variables

`postgresql_rpm_gpg_key_pgdg_x86_64`: the postgresql key. This is not needed for CentOS 7.

`server_locale`: the sets the user's language, region, etc. This is set to "en_GB.UTF-8"

`external_storage_drive`: path to mounted storage. By default this is undefined.

## Dependencies

There are no Ansible-Galaxy dependencies for this role.

## Example Playbook

This role will perform provision tasks on a managed host. To use this role, add it to the list of roles in a play:

```yaml
- name: Provision
  hosts: all
  roles:
    - mirsg.provision
```

## License

[BSD 3-Clause License](https://github.com/UCL-MIRSG/ansible-role-install-python/blob/main/LICENSE).

## Author Information

This role was created by the [Medical Imaging Research Software Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group) at [UCL](https://www.ucl.ac.uk/).
