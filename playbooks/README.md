# mirsg.infrastructure playbooks

This collection contains playbooks for deploying XNAT and setting up monitoring
of servers.

## Molecule setup

This collection is tested using Ansible Molecule.

Each playbook has its own Molecule configuration, which can be found in the
`ansible_collections/mirsg/infrastructure/playbooks/molecule/` folder.

See the top-level README for info on
[testing this collection](../README.md#testing-this-collection).

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

Create CentOS 7, RockyLinux 9, and AlmaLinux 10 scenarios for your playbook in
`ansible_collections/mirsg/infrastructure/playbooks/molecule`. See the
`mirsg.infrastructure.install_monitoring`
[CentOS 7](./molecule/centos7_monitoring/),
[RockyLinux 9](./molecule/rocky9_monitoring/), and
[AlmaLinux 10](./molecule/alma10_xnat/) scenarios for an example configuration.

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
    uses: UCL-MIRSG/.github/actions/molecule-test@v0
    with:
      tests-path: ansible_collections/mirsg/infrastructure/playbooks
```

This uses the
[`UCL-MIRSG/.github/actions/molecule-test` composite action](https://github.com/UCL-MIRSG/.github/blob/main/actions/molecule-test/action.yml)
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

## XNAT version compatibility

XNAT 1.10 is a major release that moves from Java 8 to Java 21 (and dcm4che 2
to 5) and needs newer versions of some plugins. It still runs on Tomcat 9 and
PostgreSQL 14.

`install_xnat.yml` uses the `install_java`, `tomcat`, `postgresql` and `xnat`
roles, so the versions they install must be compatible. The playbook first
checks the `xnat_version` is compatible with the `java_major_version` and
`xnat_pipeline_version`, and fails the run if the combination is not one of these:

| `xnat_version`  | `java_major_version` | `tomcat_version` | `xnat_pipeline_version` | `postgresql_version` |
| --------------- | -------------------- | ---------------- | ----------------------- | -------------------- |
| 1.8.0 – 1.9.x   | 8                    | 9.0.x            | < 1.9.0 (e.g. 1.8.10)   | 14                   |
| 1.10.0 – 1.10.1 | 21                   | 9.0.x            | >= 1.9.0 (e.g. 1.9.0)   | 14                   |

Where each variable is set:

- `xnat_version` and `xnat_pipeline_version`
  [default](../roles/xnat/defaults/main.yml) to `1.9.3.5` and `1.8.10` (the XNAT
  1.9.x profile). The assert reads these from the role defaults, so overriding
  **both** in the inventory `group_vars` for the `xnat` group is all that is
  needed to install 1.10.
- `java_major_version` [has no default](../roles/install_java/defaults/main.yml).
  It is set automatically in the [playbook group vars](./group_vars/xnat.yml)
  to ensure the correct version is installed for the selected XNAT version.
- `tomcat_version` is pinned to a 9.0.x release in the
  [playbook group vars](./group_vars/xnat.yml).
- `postgresql_version` defaults to `14` in the
  [`postgresql` role](../roles/postgresql/defaults/main.yml) and can be
  overridden in your inventory `group_vars`.
