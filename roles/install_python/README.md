# Ansible Role: mirsg.infrastructure.install_python

This role installs Python, pip, and setuptools on Debian and RedHat operating
systems. It will also update pip to the latest version or a user-specified
version, and then install user-specified Python packages using pip.

## Role Variables

`install_python` is a dictionary that contains the following variables:

`version`: the version of Python to install. This defaults to `"3"`.

`pip_version`: the version of pip to update to. This defaults to `"21.3.1"`.

`pip_executable`: path to the pip executalbe to use for installing packages.
This defaults to `"pip3"`

`system_packages`: list of system packages to be installed along with Python.
This defaults to:

```yaml
- python3
- python3-pip
- python3-setuptools
```

The packages listed in `install_python.system_packages` will be installed by the
OS package manager, NOT by pip.

`pip_packages`: list of Python packages to be installed by pip. This defaults to
`[]`.

## Example Playbook

This role will install Python on a managed host. To used this role, add it to
the list of roles in a play:

```yaml
- name: Install Python
  hosts: all
  roles:
    - mirsg.install_python
```
