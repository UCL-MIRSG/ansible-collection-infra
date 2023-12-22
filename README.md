# mirsg.infrastructure Ansible Collection

This repository contains the `mirsg.infrastructure` Ansible Collection. This
collection can be used to configure infrastructure for deploying XNAT and OMERO.

## Included content

### Playbooks

| Name                                                         | Description                                                                   |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| [setup_user_accounts.yml](playbooks/setup_user_accounts.yml) | Create OS user accounts a group of servers `target`, which defaults to `all`. |

## External requirements

Before using this collection and its playbooks, you must install the [necessary
Ansible collections and roles](meta/requirements.yml).

## Using this collection

You can install this collection using the `ansible-galaxy` command-line tool:

    ansible-galaxy collection install https://github.com/UCL-MIRSG/ansible-collection-infra.git

You can also include it in a `requirements.yml` file and install it via
`ansible-galaxy collection install -r requirements.yml` using the format:

```yaml
collections:
  - name: mirsg.infrastructure
    source: https://github.com/UCL-MIRSG/ansible-collection-infra.git
    type: git
    version: main
```

## Testing this collection

We use [Ansible Molecule](https://ansible.readthedocs.io/projects/molecule/) and
its [Docker plugin](https://github.com/ansible-community/molecule-plugins) to
test the roles and playbooks in this collection.

If you would like to run the tests locally you will need to:

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

Before running the tests you'll need to install Molecule, the Docker plugin, and
the Python Docker Engine API using `pip`:

```shell
python -m pip install molecule 'molecule-plugins[docker]' docker
```

### Testing the roles using Molecule

Molecue 6.0 requires that the test configuration is not in the top-level
directory of the collection. To support running the tests with Molecule 6, the
Molecule configuration is in `ansible_collections/mirsg/infrastructure/tests`.
To run the tests you must be in this directory:

```shell
cd ansible_collections/mirsg/infrastructure/tests
```

This collection contains [molecule
scenarios](https://ansible.readthedocs.io/projects/molecule/getting-started/#molecule-scenarios)
that allow testing of the individual roles. There are scenarios that run the
tests on both CentOS 7 and Rocky 9.

To run the CentOS 7 tests for the roles:

```shell
molecule test -s centos7_roles
```

This command will:

- install the required Ansible roles and collections
- create a CentOS 7 container
- `tests/molecule/resources/shared/prepare.yml` playbook to do any required
  setup for the roles
- run the `tests/molecule/resources/roles/converge.yml` playbook, which will run
  the roles in this collection
- run `tests/molecule/resources/roles/converge.yml` a second time to check the
  roles are
  [idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)
- run `tests/molecule/resources/roles/verify.yml`
- destroy the CentOS 7 container

### Test a single role in the collection

Individual roles in the collection can be tested by setting the
`MOLECULE_RUN_TAGS` environment variable:

```shell
export MOLECULE_RUN_TAGS=provision
molecule test -s centos7_roles
```

### Inspecting the Container

If the `molecule test` command fails at any stage, the container is immediately
destroyed. This is due to the pre-defined sequence of actions the Molecule take
when the `molecule test` command is invoked.

If you would like to be able to access the test container, you should instead
use the `molecule converge` command. To run this on CentOS 7:

```shell
molecule converge -s centos7_roles
```

This will install necessary Ansible roles and collections, create the test
container, and run the molecule playbooks. If the deployment fails, the
container is not destroyed.

#### Access the container

Once the command has finished running, you can access the container using the
name of the scenario. To access the container for the `centos7_roles` scenario:

```shell
molecule login -s centos7_roles
```

#### Destroy the container

If you use the `molecule converge` command, you must remember to destroy the
container, network, and volumes yourself. You can do this using the `molecule
destroy` command:

```shell
molecule destroy -s centos7_roles
```

### Test a playbook

Playbooks in the collection can also be tested using Molecule. An example of how
this can be done can be seen by looking at the tests for the
`mirsg.install_monitoring` playbook in this collection. This is tested on CentOS
7 and RockyLinux 9 using the
[centos7_monitoring](./tests/molecule/centos7_monitoring/) and
[rocky9_monitoring](./tests/molecule/rocky9_monitoring/) scenarios. An inventory
and associated group variables can be found in
[resources/monitoring/inventory](./tests/molecule/resources/monitoring/inventory/).
Testing the playbook also requires its own
[converge.yml](./tests/molecule/resources/monitoring/converge.yml) playbook but
it uses the shared [prepare.yml](./tests/molecule/resources/shared/prepare.yml)
playbook. Running the tests then proceeds as with testing the roles:

```shell
export MOLECULE_RUN_TAGS=monitoring
molecule test -s centos7_monitoring
```

### Integration tests

When a PR that modifies a role is opened, the changes are
[tested](.github/workflows/) by deploying that role using GitHub Actions. See
the [`molecule-firewalld` workflow](.github/workflows/molecule-firewalld.yml)
for an example.

## Code style and formatting

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

This repo has `pre-commit` hooks enabled, for instructions see
<https://github.com/UCL-MIRSG/.github/tree/main/precommit>.

## License

This collection is licensed and distributed under the BSD 3-Clause License.

## Author Information

This collection was created by the [Medical Imaging Research Software
Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group)
at [UCL](https://www.ucl.ac.uk/).
