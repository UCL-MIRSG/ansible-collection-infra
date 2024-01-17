# mirsg.infrastructure.nginx

This role is for configuring [nginx](https://www.nginx.com/) as a
[reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
on CentOS 7 or RockyLinux 9.

## Role Variables

| Name                   | Description                                                                                                                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nginx_owner`          | The OS user that will have ownership of the nginx service file and directory. Defaults to `root`                                                                                                |
| `nginx_group`          | The OS group that will have ownership of the nginx service file and directory. Defaults to `root`                                                                                               |
| `nginx_log_folder`     | The path to the nginx logs. Defaults to `/var/log/nginx`                                                                                                                                        |
| `nginx_access_log`     | File in which to write access logs for the default server. Defaults to `/var/log/access.log`                                                                                                    |
| `nginx_error_log`      | File in which to write error logs for the default server. Defaults to `/var/log/error.log`                                                                                                      |
| `nginx_app_access_log` | File in which to write access logs for the application server. Defaults to `/var/log/app.access.log`                                                                                            |
| `nginx_app_error_log`  | File in which to write error logs for the application server. Defaults to `/var/log/app.error.log`                                                                                              |
| `nginx_http_port`      | The port to listen on for HTTP connections. Defaults to `80`                                                                                                                                    |
| `nginx_https_port`     | The port to listen on for HTTPS connections. Defaults to `443`                                                                                                                                  |
| `nginx_proxy_port`     | The port to forward requests to. Required variable; no default                                                                                                                                  |
| `nginx_root`           | The path to search for static files. Optional variable; no default                                                                                                                              |
| `nginx_conf_template`  | The template to use for generating the NGINX config. See currently available [templates](templates/). Defaults to `nginx_xnat.j2`, which is used to configure NGINX as a reverse proxy for XNAT |
| `nginx_conf_file`      | The path to write the NGINX config to. Defaults to `/etc/nginx/nginx.conf`                                                                                                                      |
| `nginx_ipv6_enabled`   | Whether to enable support for IPv6. Defaults to `false`                                                                                                                                         |

If you would like to use SSL with NGINX, you will need to have the
certificate and key on your Ansible Controller, and may also need to set
the following variables:

| Name                            | Description                                                                               |
| ------------------------------- | ----------------------------------------------------------------------------------------- |
| `nginx_use_ssl`                 | Whether to use SSL. Defaults to `true`                                                    |
| `nginx_certs_dir`               | Where to store the certificates. Defaults to `/etc/nginx/ssl`                             |
| `nginx_server_cert_cache`       | Path to SSL certificate on the Ansible Controller. Required if using SSL; no default      |
| `nginx_server_key_cache`        | Path to SSL certificate on the Ansible Controller. Required if using SSL; no default      |
| `nginx_ssl_cert_file`           | Path to copy the SSL certificate to. Defaults to `/etc/nginx/ssl/server.cert`             |
| `nginx_ssl_key_file`            | Path to copy the SSL key to. Defaults to `/etc/nginx/ssl/server.key`                      |
| `nginx_diffie_helman_size_bits` | Bit size for OpenSSL Diffie-Hellman Parameters. Defaults to `4096`                        |
| `nginx_dh_params_file`          | Path to write the Diffie-Hellman Parameters to. Defaults to `"/etc/nginx/ssl/dhparam.pem` |

## Dependencies

You will need to install the following collections before using `mirsg.infrastructure.nginx`:

- `ansible.posix`
- `community.crypto`
- `community.general`

## Example Playbook

```yaml
- name: Configure nginx
  hosts: all
  roles:
    - mirsg.infrastructure.nginx
```
