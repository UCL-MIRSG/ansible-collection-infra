# mirsg.infrastructure roles

This collection contains several roles to configure infrastructure for deploying
XNAT or OMERO.

## Molecule setup

This collection is tested using Ansible Molecule.

Roles are tested at the collection level rather than as standalone roles, i.e. there is a
top-level `tests` folder for the collection rather than a `tests/` folder for each role. This
is to avoid duplicating the Ansible Molecule configuration for each role (especially the
scenario definitions).

There is a [single converge playbook](../tests/molecule/resources/converge.yml) that by default
will run all the roles in the playbook. If you would like to run `molecule converge` on a specific
role instead, you can do this by setting the relevant Ansible tags.

The tags can be set using the
`MOLECULE_RUN_TAGS` environment variable, which is passed to Ansible Molecule in the
[scenario definition](../tests/molecule/centos7/molecule.yml) (see the
`config_options` for the `provisioner` section of the scenario). The accepted tags are defined in the
[converge playbook](../tests/molecule/resources/converge.yml) for each role.

As an example, to run
`molecule converge` on the `provision` role only:

```bash
export MOLECULE_RUN_TAGS=provision
molecule converge -s centos7
```

## Adding a new role

To add a new role to this collection, you will need to:

1. add the role to the `roles/` folder
2. add a molecule configuration for the role to the `tests/` folder
3. add a workflow for the role to the `.github/workflows` folder

### Add a new role to the roles folder

You can use the `ansible-galaxy init` command to create a new role in the `roles/` folder:

```bash
ansible-galaxy init my_role
```

Note, roles within a collection cannot contain hyphens in their names - please use underscores
instead.

You can delete the `meta` and `tests` folders as these are handled at the collection level
rather than for individual roles:

```bash
rm -r my_role/meta my_role/tests
```

You can then add your tasks, defaults, etc. to this role as usual.

### Add a molecule configuration

We will want to be able to run `molecule converge` on the new role. To do this, add
the role to the list of roles in the [converge playbook](../tests/molecule/resources/converge.yml):

```yaml
- role: mirsg.infrastructure.my_role
  tags: my_role
```

We have added a tag `my_role` so that we can optionally tell molecule to only include this role
in the tests (see [above](#molecule-setup) for more info). To run molecule only on this new role,
you will need to set the `MOLECULE_RUN_TAGS` environment variable:

```bash
export MOLECULE_RUN_TAGS=my_role
molecule converge -s centos7
```

If necessary, add [inventory `group_vars`](../tests/molecule/resources/inventory/group_vars)
for your role.

You may also need to add [`prepare`](../tests/molecule/resources/prepare.yml) or
[`verify`](../tests/molecule/resources/verify.yml) steps for your role.

### Add a GitHub Workflow

We have a GitHub workflow for running molecule on each role individually, and doing so
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
  molecule-firewalld:
    uses: ./.github/workflows/molecule.yml
    with:
      run-tags: my_role
```

This uses the [`.github/workflows/molecule.yml` reusable workflow](.github/workflows/molecule.yml) to run
molecule on the collection. Specifying `run-tags: my_role` will set `MOLECULE_RUN_TAGS` to `my_role` in the
reusable workflow, and so only `mirsg.infrastructure.my_role` will be the only role ran during the tests.
