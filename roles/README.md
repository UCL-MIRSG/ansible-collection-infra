# mirsg.infrastructure roles

This collection contains several roles to configure infrastructure for deploying
XNAT or OMERO.

## Molecule setup

This collection is tested using Ansible Molecule.

Each role has its own Molecule configuration, which can be found it the `molecule/`
folder of each role. Molecule base configurations are used to reduce to amount
of duplication in the setup for testing each role. There are two base configurations
that correspond to two [Molecule
scenarios](https://ansible.readthedocs.io/projects/molecule/getting-started/#molecule-scenarios)
one for testing on CentOS 7 and another for testing on RockyLinux 9. The base configurations
are in the `ansible_collections/mirsg/infrastructure/tests` folder.

## Adding a new role

To add a new role to this collection, you will need to:

1. add the role to the `roles/` folder
2. add a molecule configuration for the role to the `roles/<my_role>/molecule/` folder
3. add a workflow for the role to the `.github/workflows` folder

### Add a new role to the roles folder

You can use the `ansible-galaxy init` command to create a new role in the `roles/` folder:

```bash
ansible-galaxy init my_role
```

Note, roles within a collection cannot contain hyphens in their names - please use underscores
instead.

You can delete the `meta` and `tests` folders as these metadata is stored at the collection level
and the molecule configuration should be stored in a `molecule/` folder:

```bash
rm -r my_role/meta my_role/tests
```

You can then add your tasks, defaults, etc. to this role as usual.

### Add a molecule configuration

Create a `molecule` folder in your role:

```bash
mkdir my_roles/molecule
```

Add the `converge.yml` playbook to the `molecule/` folder,
and create folders for the `centos7` and `rocky9` scenarios. See
[`mirsg.infrastructure.firewalld` for an example](./firewalld/).

If necessary, add inventory `group_vars` under a `resources` subfolder: `my-role/molecule/resources/inventory/group_vars`.
for your role.

You may also need to add `prepare` or
`verify` playbooks for your role. These can be added to the `resources` subfolder if you have multiple scenarios that will share the playbooks, or in the default scenario folder.

### Add a GitHub Workflow

We have a GitHub workflow for running molecule on each role, and doing so
only when that role changes.

You should add a GitHub workflow for your new role to `.github/workflows/molecule-my-role.yml`.
The workflow should follow this format:

```yaml
name: Test my_role
on:
  pull_request:
    paths:
      - "roles/my_role/**"
      - ".github/workflows/molecule.yml"
      - ".github/workflows/molecule-my-role.yml"

jobs:
  molecule-my_role:
    uses: ./.github/workflows/molecule.yml
    with:
      tests-path: ansible_collections/mirsg/infrastructure/roles/my_role
```

This uses the [`.github/workflows/molecule.yml` reusable workflow](.github/workflows/molecule.yml) to run
molecule on the relevant role.
