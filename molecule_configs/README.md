# mirsg.infrastructure Molecule base configurations

This folder contains base configurations for running tests with [Ansible
Molecule](https://ansible.readthedocs.io/projects/molecule/).

The base configuration is merged with - and can be overridden by - the scenario
configuration. To use a base configuration, specify the path to one of the files
in this directory. For example, to use the CentOS 7 base configuration:

```
molecule --base-config <path to molecule_configs>/centos7_base_config.yml
```
