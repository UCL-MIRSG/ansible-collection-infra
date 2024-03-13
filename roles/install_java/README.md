# Ansible Role: mirsg.infrastructure.install_java

A role to install and configure Java.

## Role Variables

`java_profile_d`: Defaults to "/etc/profile.d".

`java_home`: Defaults to "/usr/lib/jvm/jre".

`java_package`: Defaults to "java-1.8.0".

`java_package_version`: Defaults to "devel".

## Example Playbook

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  roles:
    - { role: mirsg.infrastructure.install_java }
```
