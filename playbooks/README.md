# mirsg.infrastructure playbooks

This collection contains playbooks for deploying XNAT and setting up monitoring
of servers.

## Molecule setup

This collection is tested using Ansible Molecule.

Each playbook has its own Molecule configuration, which can be found in the
`ansible_collections/mirsg/infrastructure/playbooks/molecule/` folder. Molecule
base configurations are used to reduce to amount of duplication in the setup for
testing each playbook. There are two base configurations that correspond to two
[Molecule scenarios](https://ansible.readthedocs.io/projects/molecule/getting-started/#molecule-scenarios)
one for testing on CentOS 7 and another for testing on RockyLinux 9. The base
configurations are in the
`ansible_collections/mirsg/infrastructure/molecule_configs` folder.

## Adding a new playbook

To add a new playbook to this collection, you will need to:

1. add the playbook to the `playbooks/` folder
2. add a molecule configuration for the playbook to the `playbooks/molecule/`
   folder
3. add a workflow for the playbook to the `.github/workflows` folder

### Add a new playbook to the playbooks folder

The playbooks in an Ansible Collection are the same as any other playbook - no
special setup is required. If you create a playbook
`ansible_collections/mirsg/infrastructure/playbooks/my_playbook.yml`, you will
be able to run the playbook using `ansible-playbook`:

```bash
ansible-playbook mirsg.infrastructure.my_playbook
```

or include it within another playbook:

```yaml
- name: Run my_playbook
  ansible.builtin.import_playbook: mirsg.infrastructure.my_playbook
```

### Add a molecule configuration

Create CentOS 7 and RockyLinux 9 scenarios for your playbook in
`ansible_collections/mirsg/infrastructure/playbooks/molecule`. See the
`mirsg.infrastructure.install_monitoring`
[CentOS 7](./molecule/centos7_monitoring/) and
[RockyLinux 9](./molecule/rocky9_monitoring/) scenarios for an example
configuration.

Add the `converge.yml` playbook to a shared `resources` subfolder:
`molecule/resources/my_playbook/converge.yml`.

If necessary, add inventory `group_vars` to the shared `resources` subfolder:
`molecule/resources/my_playbook/inventory/group_vars`. for your playbook.

You may also need to add `prepare` or `verify` playbooks. These can be added to
the the same shared `resources` folder.

You will need to update the `molecule.yml` configuration file with the correct
paths to these playbooks.

### Add a GitHub Workflow

You should add a GitHub workflow for your new playbook to
`.github/workflows/molecule-my-playbook.yaml`. The workflow should follow this
format:

```yaml
name: Test my_playbook
on:
  pull_request:
    paths:
      - "playbooks/my_playbook.yml"
      - ".github/workflows/molecule-role-used-in-playbook.yaml"

jobs:
  molecule-my_playbook:
    uses: ./.github/workflows/molecule.yaml
    with:
      tests-path: ansible_collections/mirsg/infrastructure/playbooks
```

This uses the
[`.github/workflows/molecule.yaml` reusable workflow](.github/workflows/molecule.yaml)
to run molecule on the relevant role.

## Playbooks

| Playbook                                                 | Description                                                                                                                                                                                                                                                                    |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [install_monitoring.yml](./install_monitoring.yml)       | Playbook for installing Dockerised deployment of Prometheus on the monitoring server and `node_exporter` on each of the client servers to be monitored.                                                                                                                        |
| [install_omero.yml](./install_omero.yml)                 | Playbook for installing OMERO.server and OMERO.web along with Postgresql in a separate host.                                                                                                                                                                                   |
| [install_xnat.yml](./install_xnat.yml)                   | Playbook for installing XNAT along with Postgresql in a separate host.                                                                                                                                                                                                         |
| [restart_xnat_or_omero.yml](./restart_xnat_or_omero.yml) | Playbook for restarting XNAT or OMERO (as well as the associated Postgresql service). To be used in the event of a system outage. To run a restart for OMERO make sure to set the `services_to_restart` variable when calling the playbook to `['omero-server', 'omero-web']`. |
| [setup_user_accounts.yml](./setup_user_accounts.yml)     | Playbook for adding MIRSG administrator accounts to the application and database hosts.                                                                                                                                                                                        |
| [upgrade_postgresql.yml](./upgrade_postgresql.yml)       | Playbook for running an upgrade of Postgresql.                                                                                                                                                                                                                                 |
