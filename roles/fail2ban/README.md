# mirsg.infrastructure.fail2ban

This role installs [fail2ban](https://github.com/fail2ban/fail2ban) on CentOS 7,
or on EL8 and later (e.g. Rocky Linux, AlmaLinux), to ban addresses that
misbehave, e.g. that keep hitting the nginx rate limit set by
`mirsg.infrastructure.nginx`.

Addresses are banned on all ports, before firewalld's rules, so an address is
banned whichever zone its traffic is in, e.g. a source added to the `work` zone.
Run the role after `mirsg.infrastructure.firewalld`. How bans are added depends
on the OS:

- On EL8 and later, fail2ban adds bans to its own nftables table, `f2b-table`.
  firewalld leaves the table alone, so bans stay in place when firewalld is
  reloaded or restarted. See them with `nft list table inet f2b-table`.
- On CentOS 7, where firewalld uses iptables, fail2ban adds bans to a kernel
  ipset that a firewalld direct rule drops traffic from. See them with
  `ipset list`. The direct rule is a runtime rule, so it's lost when firewalld
  is reloaded. The role makes fail2ban restart whenever firewalld restarts, to
  add it again, but `firewall-cmd --reload` doesn't restart the service, so run
  `systemctl restart fail2ban` after it.

Either way, bans don't show in `firewall-cmd --list-all`. Use
`fail2ban-client status <jail>` to see them.

The role checks the config with `fail2ban-client --test` before restarting
fail2ban, and fails if any jail isn't running.

## Jails

Set `fail2ban_jails` to the jails to enable. Each jail needs a `name`, and any
other keys are written to the jail's section of the config. A jail uses the
filter with the same name unless it sets `filter`. For example, the jails used
for XNAT are:

```yaml
fail2ban_jails:
  # Clients that keep hitting the nginx rate limit
  - name: nginx-limit-req
    logpath: /var/log/nginx/xnat.error.log
  # Wrong passwords for the REST API
  - name: nginx-proxy-basic-auth
    logpath: /var/log/nginx/xnat.access.log
    maxretry: 10
  # Scanners looking for e.g. /wp-login.php
  - name: nginx-botsearch
    logpath: /var/log/nginx/xnat.access.log
```

`nginx-limit-req` and `nginx-botsearch` are provided by fail2ban.
`nginx-limit-req` doesn't match the messages nginx logs in its dry run mode
(`nginx_rate_limit_dry_run`), so no one is banned until the rate limit is on.

The role adds the filters in [files/filter.d](files/filter.d):

- `nginx-proxy-basic-auth` matches `401` responses in an nginx access log, in
  the default `combined` format, to requests that sent HTTP basic auth
  credentials. nginx logs the username as `$remote_user`, so requests without
  credentials, e.g. from a logged out browser tab, don't match.

## Role Variables

| Name                         | Description                                                                                                                                                     |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fail2ban_jails`             | Jails to enable, see [Jails](#jails). Defaults to `[]`                                                                                                          |
| `fail2ban_ignoreip`          | Addresses or CIDR ranges that are never banned, in addition to localhost. Defaults to `[]`                                                                      |
| `fail2ban_bantime`           | How long to ban an address for. Defaults to `1h`                                                                                                                |
| `fail2ban_findtime`          | The window in which `maxretry` matches cause a ban. Defaults to `10m`                                                                                           |
| `fail2ban_maxretry`          | How many matches within `findtime` cause a ban. Jails can set their own. Defaults to `5`                                                                        |
| `fail2ban_bantime_increment` | Whether to double the ban each time an address is banned again. Defaults to `true`                                                                              |
| `fail2ban_bantime_maxtime`   | The longest ban. Bans are remembered for this long, so repeat offenders' bans keep increasing. Defaults to `1w`                                                 |
| `fail2ban_banaction`         | The action used to ban addresses. Defaults to `nftables-allports` on EL8 and later, and `firewallcmd-ipset[actiontype=<allports>, ipsettype=ipset]` on CentOS 7 |

## Useful commands

```sh
# List the jails, and the addresses banned by a jail
fail2ban-client status
fail2ban-client status nginx-limit-req

# Unban an address, or every address
fail2ban-client unban 192.0.2.1
fail2ban-client unban --all

# Test a filter against a log
fail2ban-regex /var/log/nginx/xnat.access.log nginx-proxy-basic-auth
```

## Dependencies

fail2ban is installed from [EPEL](https://docs.fedoraproject.org/en-US/epel/),
which `mirsg.infrastructure.provision` installs.

## Example Playbook

```yaml
- name: Ban addresses that keep hitting the rate limit
  hosts: all
  become: true
  roles:
    - role: mirsg.infrastructure.firewalld
    - role: mirsg.infrastructure.fail2ban
      vars:
        fail2ban_jails:
          - name: nginx-limit-req
            logpath: /var/log/nginx/app.error.log
```
