# Ansible Role: mirsg.provision_accounts

This role creates OS users on provisioned infrastructure.

## Role Variables

There are several variables that **must be set** to use this role.

`os_users`: A list of OS users to create. Each user definition is dictionary
containing:

- `username`: The name of the user
- `crypted_password`: Encrypted user password.

`os_user_groups`: OS groups the user should belong to expressed as a single
string with commas separating each group.

For details on how to generated encrypted passwords, see:
https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module

## Dependencies

There are no Ansible-Galaxy dependencies for this role.

## Example Playbook

This role will perform provision tasks on a managed host. To use this role, add
it to the list of roles in a play:

```yaml
- name: Provision accounts
  hosts: all
  roles:
    - mirsg.infrastructure.provision_accounts
```
