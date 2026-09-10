# Ansible Role: mirsg.infrastructure.install_java

A role to install and configure Java using
[Adoptium's Eclipse Temurin](https://adoptium.net/).

## Role Variables

`java_profile_d`: Defaults to "/etc/profile.d".

`java_major_version`: The major version of Java to install. **Required**: no
default, so callers must set it explicitly.

`java_package`: The Eclipse Temurin package name, derived from
`java_major_version` (e.g. `temurin-8-jdk`).

`java_home`: The JDK's install directory. Defaults to `/usr/lib/jvm/jre`.

`java_adoptium_repo_baseurl` / `java_adoptium_repo_gpgkey`: The Temurin yum
repository and GPG key used to install the package.

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: mirsg.infrastructure.install_java
      vars:
        java_major_version: 21
```
