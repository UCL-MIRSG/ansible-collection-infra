# mirsg.infrastructure Ansible Collection

This repository contains the `mirsg.infrastructure` Ansible Collection. This
collection can be used to configure infrastructure for deploying XNAT and OMERO.

## Included content

### Playbooks

| Name                                                         | Description                                                                                   |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| [setup_user_accounts.yml](playbooks/setup_user_accounts.yml) | Create OS user accounts a group of servers `target`, which defaults to `all`.                 |
| [install_monitoring.yml](playbooks/install_monitoring.yml)   | Configure a host to collect metrics from client machines.                                     |
| [install_xnat.yml](playbooks/install_xnat.yml)               | Install XNAT on two-tier infrastructure with XNAT Container Service (Docker host and client). |

## External requirements

Before using this collection and its playbooks, you must install the
necessary Ansible collections and roles, choosing the requirements file appropriate for your environment (e.g., [EL7](meta/el7-requirements.yml) or [EL9](meta/el9-requirements.yml)).

## Using this collection

This collection can be installed using the `ansible-galaxy` command-line tool:

ansible-galaxy collection install
<https://github.com/UCL-MIRSG/ansible-collection-infra.git>

It can also be included in a `requirements.yml` file and install it via
`ansible-galaxy collection install -r requirements.yml` using the format:

```yaml
collections:
  - name: mirsg.infrastructure
    source: https://github.com/UCL-MIRSG/ansible-collection-infra.git
    type: git
    version: main
```

## Using the playbooks

N.B. Ideally, changes to the infrastructure should be made via workflows.
However, in some cases it may be necessary to run the playbooks manually. For
example, for upgrading the postgresql version on the `ucl_test` server:

```shell
ansible-playbook mirsg.infrastructure.upgrade_postgresql -u <user> --ask-become-pass -l ucl_test -i environments/cs-production/xnat/hosts.yml --vault-password-file ~/.ucl_xnat-vault_password
```

The [upgrade_postgresql.yml](./playbooks/upgrade_postgresql.yml) playbook
prompts for the version to upgrade from and to upgrade to.

The following variable needs to be set:
`postgresql_upgrade_data_dir: “/var/lib/pgsql/{{ postgreql_upgrade_new_version }}/data”`

This is done in the group vars for the server you're updating i.e.
[ucl_test](https://github.com/UCL-MIRSG/UCLMedicalImagingEnv/blob/main/environments/cs-production/xnat/group_vars/ucl_test/vars)

## Testing this collection

[Ansible Molecule](https://ansible.readthedocs.io/projects/molecule/) and its
[Docker plugin](https://github.com/ansible-community/molecule-plugins) is used
to test the roles and playbooks in this collection.

To run the tests locally you will need to:

- clone this repository
- install Ansible Molecule and other test requirements
- install and start the [Docker Engine](https://docs.docker.com/engine/install/)
  or [Docker Desktop](https://www.docker.com/products/docker-desktop/) on MacOS,
  if you have not done so already
- if necessary (i.e. you are running Docker on Linux), add your user to the
  docker group in order to access the Docker Engine
- run the tests using Molecule

### Clone this repository

To test a collection, Molecule requires that it is in the path
`ansible_collections/<namespace>/<collection name>`. This means when you clone
this repository you must ensure it is in the path
`ansible_collections/mirsg/infrastructure`. The simplest way to do this is:

```shell
git clone git@github.com:UCL-MIRSG/ansible-collection-infra.git ansible_collections/mirsg/infrastructure
```

### Install Ansible Molecule

Before running the tests install Molecule, the Docker plugin, and the Python
Docker Engine API using `pip`:

```shell
python -m pip install molecule 'molecule-plugins[docker]' docker
```

### Testing the roles using Molecule

Each role has its own Molecule configuration, which can be found it the
`molecule/` folder of each role. Molecule base configurations are used to reduce
to amount of duplication in the setup for testing each role. There are two base
configurations that correspond to two
[Molecule scenarios](https://ansible.readthedocs.io/projects/molecule/configuration/#scenario)
one for testing on CentOS 7 and another for testing on RockyLinux 9. The base
configurations are in the
`ansible_collections/mirsg/infrastructure/molecule_configs` folder.

To run the tests for a specific role, first navigate the the role directory,
e.g.

```shell
cd ansible_collections/mirsg/infrastructure/roles/provision
```

Then invoke Molecule with a base config and corresponding scenario. To run the
tests on CentOS 7:

```shell
molecule --base-config ../../molecule_configs/centos7_base_config.yml test --scenario centos7
```

This command will:

- install the required Ansible roles and collections
- create a CentOS 7 container
- run the `molecule/prepare.yml` playbook (if it exists) to do any required
  setup for the role
- run the `molecule/converge.yml` playbook, which will run the role being tested
- run `molecule/converge.yml` a second time to check the role is
  [idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)
- run `molecule/verify.yml` playbook (if it exists) to perform verification
- destroy the CentOS 7 container

To run the tests on Rocky 9 instead:

```shell
molecule --base-config ../../molecule_configs/rocky9_base_config.yml test --scenario rocky9
```

### Inspecting the Container

If the `molecule test` command fails at any stage, the container is immediately
destroyed. This is due to the pre-defined sequence of actions the Molecule take
when the `molecule test` command is invoked.

If you would like to be able to access the test container, you should instead
use the `molecule converge` command. To run this on CentOS 7:

```shell
molecule --base-config ../../molecule_configs/centos7_base_config.yml converge --scenario centos7
```

This will install necessary Ansible roles and collections, create the test
container, and run the molecule playbooks. If the deployment fails, the
container is not destroyed.

#### Access the container

Once the command has finished running, you can access the container using the
name of the scenario. To access the container for the `centos7` scenario:

```shell
molecule --base-config ../../molecule_configs/centos7_base_config.yml login --scenario centos7
```

If testing a role or playbook where Molecule creates multiple containers,
individual hosts can be accessed using the `--host` flag:

```shell
molecule --base-config ../../molecule_configs/centos7_base_config.yml login --scenario centos7_monitoring --host mserv
```

#### Destroy the container

If you use the `molecule converge` command, you must remember to destroy the
container, network, and volumes yourself. You can do this using the
`molecule destroy` command:

```shell
molecule --base-config ../../molecule_configs/centos7_base_config.yml  destroy --scenario centos7
```

### Test a playbook

Playbooks in the collection can also be tested using Molecule. The Molecule
configuration for playbooks is in the
`ansible_collections/mirsg/infrastructure/playbooks/molecule` folder.

An example of how to setup testing for a playbook can be seen by looking at the
tests for the `mirsg.install_monitoring` playbook in this collection. This is
tested on CentOS 7 and RockyLinux 9 using the
[centos7_monitoring](./playbooks/molecule/centos7_monitoring/) and
[rocky9_monitoring](./playbooks/molecule/rocky9_monitoring/) scenarios.

An inventory and associated group variables can be found in
[playbooks/molecule/resources/monitoring/inventory](./playbooks/molecule/resources/monitoring/inventory/).
Testing the playbook also requires its own
[converge.yml](./playbooks/molecule/resources/monitoring/converge.yml), and
optional `prepare.yml` and `verify.yml` playbooks. Running the tests then
proceeds as with testing the roles:

```shell
molecule test --scenario centos7_monitoring
```

Alternative you can use `molecule converge` to avoid destroying the container,
as [described above](#inspecting-the-container).

> [!TIP] If you run `molecule converge` on XNAT or OMERO scenarios, you can
> access the XNAT web UI at `http://localhost:8080` and the OMERO web UI at
> `http://localhost:8080/webclient`.

### Integration tests

When a PR that modifies a role or playbook is opened, the changes are
[tested](.github/workflows/) by deploying that role using GitHub Actions. See
the [`molecule-firewalld` workflow](.github/workflows/molecule-firewalld.yaml)
for an example.

## Code style and formatting

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

This repo has `pre-commit` hooks enabled, for instructions see
<https://github.com/UCL-MIRSG/.github/tree/main/precommit>.

## License

This collection is licensed and distributed under the BSD 3-Clause License.

## Author Information

This collection was created by the
[Medical Imaging Research Software Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group)
at [UCL](https://www.ucl.ac.uk/).
