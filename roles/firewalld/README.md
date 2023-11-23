# Role Name

This role configures firewalld for use in a dual VM deployment (web app VM +
database VM) of XNAT or OMERO.

## Role Variables

See `defaults/main.yml` for the full list.

- `allow_public_access`: Allow access from an IP address. Defaults to `false`.
- `internal_zone_open_services`: A list of services to allow in the `internal`
  zone. Defaults to:
  ```yaml
  - http
  - https
  - ssh
  ```
- `public_zone_open_services`: A list of services to allow in the `public` zone.
  Defaults to:
  ```yaml
  - http
  - https
  ```
- `work_zone_open_services`: A list of services to allow in the `work` zone.
  Defaults to:
  ```yaml
  - http
  - https
  ```
- `internal_zone_closed_services`: A list of services to not allow in the
  `internal` zone. Defaults to:
  ```yaml
  - samba-client
  ```
- `public_zone_closed_services`: A list of services to not allow in the `public`
  zone. Defaults to:
  ```yaml
  - ssh
  ```
- `work_zone_closed_services`: A list of services to not allow in the `work`
  zone. Defaults to:
  ```yaml
  - ssh
  ```
- `internal_zone_sources`: A list of IP addresses to allow in `internal` zone.
  Defaults to `[]`.
- `public_zone_sources`: A list of IP addresses to allow in `public` zone.
  Defaults to `[]`.
- `work_zone_sources`: A list of IP addresses to allow in `work` zone. Defaults
  to `[]`.
- `internal_zone_ports`: A list of ports to allow in `internal` zone. Defaults
  to `[]`.
- `work_zone_ports`: A list of ports to allow in `public` zone. Defaults to
  `[]`.
- `public_zone_ports`: A list of ports to allow in `work` zone. Defaults to
  `[]`.
- `rich_rules`: A list of hashes defining rich rules to apply. The zone to apply
  the rule to should be a key in the hash.

## Installation

Include in a `requirements.yml` file as follows:

```yaml
- src: https://github.com/UCL-MIRSG/ansible-role-dual-vm-firewalld.git
  version: 2022.12.22.0
  name: mirsg.firewalld
```

## Example Playbook

```yaml
- hosts: servers
  roles:
    - { role: mirsg.firewalld }
```

## License

[BSD 3-Clause License](https://github.com/UCL-MIRSG/ansible-role-postgresql/blob/main/LICENSE).

## Author Information

This role was created by the [Medical Imaging Research Software Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group) at [UCL](https://www.ucl.ac.uk/).
