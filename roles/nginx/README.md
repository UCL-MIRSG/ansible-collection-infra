# mirsg.infrastructure.nginx

This role is for configuring [nginx](https://www.nginx.com/) as a
[reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
on CentOS 7, or on EL8 and later (e.g. Rocky Linux, AlmaLinux).

The role installs the nginx mainline version pinned in `nginx_container_image`:

- on EL8 and later, from the
  [nginx.org](https://nginx.org/en/linux_packages.html) mainline repository. Any
  nginx packages from the OS or EPEL are removed first.
- on CentOS 7, which nginx.org no longer builds packages for, by running the
  official [nginx image](https://hub.docker.com/_/nginx) with Docker, using the
  host network. Docker is installed and started with
  `mirsg.infrastructure.docker` (`tasks_from: install.yml` and `start.yml`). The
  Docker daemon config, certificates and swarm are not changed, so apply the
  `mirsg.infrastructure.docker` role separately if the host needs them. An
  existing nginx service is stopped and disabled, but its package is not
  removed. The role writes `/etc/logrotate.d/nginx`, replacing the config from
  the EPEL package, so the container reopens its logs after rotation.

To go back to the EPEL nginx package on CentOS 7, remove the container and
restore the package's logrotate config before running the previous release of
this collection:

```sh
docker rm --force nginx
# rpm keeps a modified config file on reinstall, so remove it first
pkg=$(rpm -qf --queryformat '%{NAME}' /etc/logrotate.d/nginx)
rm /etc/logrotate.d/nginx
yum reinstall "$pkg"
```

The role fails if the running nginx isn't `nginx_version`.

## Role Variables

| Name                         | Description                                                                                                                                                                                                                                              |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nginx_install_method`       | `package` to install from the nginx.org repository, or `container` to run the official nginx image. Defaults to `container` on EL7 and `package` otherwise                                                                                               |
| `nginx_container_image`      | The nginx image to run, pinned by tag and digest. Defaults to the latest mainline release when the role was released, e.g. `nginx:1.31.9@sha256:...`                                                                                                     |
| `nginx_version`              | The nginx version to install from nginx.org. Defaults to the version in the `nginx_container_image` tag, so both install methods run the same nginx                                                                                                      |
| `nginx_repo_baseurl`         | The nginx.org repository to install from. Defaults to the mainline repository, `https://nginx.org/packages/mainline/rhel/$releasever/$basearch/`                                                                                                         |
| `nginx_repo_gpgkey`          | The key used to sign the nginx.org packages. Defaults to `https://nginx.org/keys/nginx_signing.key`                                                                                                                                                      |
| `nginx_container_name`       | Name of the nginx container. Defaults to `nginx`                                                                                                                                                                                                         |
| `nginx_http2`                | Whether to enable HTTP/2. Defaults to `true`                                                                                                                                                                                                             |
| `nginx_owner`                | The OS user that will have ownership of the nginx service file and directory. Defaults to `root`                                                                                                                                                         |
| `nginx_group`                | The OS group that will have ownership of the nginx service file and directory. Defaults to `root`                                                                                                                                                        |
| `nginx_log_folder`           | The path to the nginx logs. Defaults to `/var/log/nginx`                                                                                                                                                                                                 |
| `nginx_access_log`           | File in which to write access logs for the default server. Defaults to `/var/log/access.log`                                                                                                                                                             |
| `nginx_error_log`            | File in which to write error logs for the default server. Defaults to `/var/log/error.log`                                                                                                                                                               |
| `nginx_app_access_log`       | File in which to write access logs for the application server. Defaults to `/var/log/app.access.log`                                                                                                                                                     |
| `nginx_app_error_log`        | File in which to write error logs for the application server. Defaults to `/var/log/app.error.log`                                                                                                                                                       |
| `nginx_http_port`            | The port to listen on for HTTP connections. Defaults to `80`                                                                                                                                                                                             |
| `nginx_https_port`           | The port to listen on for HTTPS connections. Defaults to `443`                                                                                                                                                                                           |
| `nginx_proxy_port`           | The port to forward requests to. Required variable; no default                                                                                                                                                                                           |
| `nginx_root`                 | The path to search for static files. Optional variable; no default                                                                                                                                                                                       |
| `nginx_conf_template`        | The template to use for generating the NGINX config. See currently available [templates](templates/). Defaults to `nginx_reverse_proxy.j2`, which is used to configure NGINX as a reverse proxy                                                          |
| `nginx_conf_file`            | The path to write the NGINX config to. Defaults to `/etc/nginx/nginx.conf`                                                                                                                                                                               |
| `nginx_add_default_server`   | Whether to add an additional server block for a default server that returns an empty response. Defaults to `true`                                                                                                                                        |
| `nginx_ipv6_enabled`         | Whether to enable support for IPv6. Defaults to `false`                                                                                                                                                                                                  |
| `nginx_rate_limit_paths`     | Paths to rate limit POST and PUT requests to, per client IP, e.g. `/login`. Sub-paths and `;jsessionid` parameters are included, and matching is case-insensitive. Only used by `nginx_reverse_proxy.j2`. Defaults to `[]`, which disables rate limiting |
| `nginx_rate_limit_rate`      | The rate to allow requests to `nginx_rate_limit_paths` at. Defaults to `6r/m`                                                                                                                                                                            |
| `nginx_rate_limit_burst`     | How many requests over the rate to allow before responding with `429 Too Many Requests`. Defaults to `5`                                                                                                                                                 |
| `nginx_rate_limit_allowlist` | Addresses or CIDR ranges that are never rate limited, e.g. a NAT gateway many users share. Defaults to `[]`                                                                                                                                              |
| `nginx_rate_limit_dry_run`   | Whether to only log requests that would be rate limited, without rejecting them. Defaults to `false`                                                                                                                                                     |

If you would like to use SSL with NGINX, you will need to have the certificate
and key on your Ansible Controller, and may also need to set the following
variables:

| Name                            | Description                                                                               |
| ------------------------------- | ----------------------------------------------------------------------------------------- |
| `nginx_use_ssl`                 | Whether to use SSL. Defaults to `true`                                                    |
| `nginx_certs_dir`               | Where to store the certificates. Defaults to `/etc/nginx/ssl`                             |
| `nginx_server_cert_cache`       | Path to SSL certificates on the Ansible host. Required if using SSL; no default           |
| `nginx_server_key_cache`        | Path to SSL certificate on the Ansible host. Required if using SSL; no default            |
| `nginx_ssl_cert_file`           | Path to copy the SSL certificate to. Defaults to `/etc/nginx/ssl/server.cert`             |
| `nginx_ssl_key_file`            | Path to copy the SSL key to. Defaults to `/etc/nginx/ssl/server.key`                      |
| `nginx_diffie_helman_size_bits` | Bit size for OpenSSL Diffie-Hellman Parameters. Defaults to `4096`                        |
| `nginx_dh_params_file`          | Path to write the Diffie-Hellman Parameters to. Defaults to `"/etc/nginx/ssl/dhparam.pem` |

## Dependencies

You will need to install the following collections before using
`mirsg.infrastructure.nginx`:

- `ansible.posix`
- `community.crypto`
- `community.docker` (CentOS 7 only)
- `community.general`

## Example Playbook

```yaml
- name: Configure nginx
  hosts: all
  roles:
    - mirsg.infrastructure.nginx
```
