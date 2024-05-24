# mirsg.docker

This role is for installing [docker-ce](https://docs.docker.com/engine/install/)
on CentOS 7 or Rocky Linux 8.

## Role Variables

| Name                       | Description                                                                                          |
| -------------------------- | ---------------------------------------------------------------------------------------------------- |
| `docker_version`           | The version of Docker to install. Defaults to `latest`.                                              |
| `docker_owner`             | The OS user that will have ownership of the Docker service file and directory. Defaults to `root`    |
| `docker_group`             | The OS group that will have ownership of the Docker service file and directory. Defaults to `root`   |
| `docker_service_directory` | The path to the Docker service. Defaults to `/etc/systemd/system/docker.service.d`                   |
| `docker_service_name`      | The name of the Docker service. Defaults to `docker`                                                 |
| `docker_rpm_gpg_key_url`   | The url of the Docker repository GPG key. Defaults to `https://download.docker.com/linux/centos/gpg` |
| `docker_repo_baseurl`      | URL to the directory containing the repodata. Defaults to `https://download.docker.com/linux/centos` |
| `docker_yum_package`       | The name of the Docker package. Defaults to `docker`                                                 |

If you would like to
[configure](https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket)
your Docker server such that clients can connect to it via TLS, you can also use
this role to generate the necessary certificates. The following variables can be
used to configure certificate creation and signing:

| Name                                        | Description                                                                                                                                    |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `docker_generate_certificates`              | If `true`, CA, server, and client certificates will be generated. Defaults to `false`                                                          |
| `docker_certificate_directory`              | Directory in which to store the certificates. Defaults to `/home/docker/.docker`                                                               |
| `docker_config_dir`                         | Docker configuration directory. Defaults to `/etc/docker`                                                                                      |
| `docker_daemon_conf_file`                   | Docker daemon configuration filename. Defaults to `/etc/docker/daemon.json`                                                                    |
| `docker_server_hostname`                    | Hostname of your Docker server. Used for the `commonName` field of the certificate signing request subject. Defaults to `"{{ ansible_host }}"` |
| `docker_server_ip`                          | IP address of your Docker server. Defaults to `0.0.0.0`                                                                                        |
| `docker_server_port`                        | Port the Docker Daemon will listen on. Defaults to `2376`.                                                                                     |
| `docker_tls_verify`                         | If `true`, require that TLS certificates can be verified by a root authority. Defaults to `true`                                               |
| `docker_ca_key`                             | Filename for the CA certificate key. Defaults to `/home/docker/.docker/ca.key`                                                                 |
| `docker_ca_csr`                             | Filename for the CA certificate signing request. Defaults to `/home/docker/.docker/ca.csr`                                                     |
| `docker_ca_cert`                            | Filename for the CA certificate. Defaults to `/home/docker/.docker/ca.pem`                                                                     |
| `docker_server_key`                         | Filename for the server certificate key. Defaults to `/home/docker/.docker/server-key.pem`                                                     |
| `docker_server_csr`                         | Filename for the server certificate signing request. Defaults to `/home/docker/.docker/server.csr`                                             |
| `docker_server_cert`                        | Filename for the server certificate. Defaults to `/home/docker/.docker/server-cert.pem`                                                        |
| `docker_client_hostnames`                   | List of hostnames of clients that will connect to the server. Defaults to `[]`                                                                 |
| `docker_client_certificate_directory`       | Directory in which to store the client certificates. Defaults to `/home/docker/.docker/client_certs`                                           |
| `docker_client_certificate_cache_directory` | Directory in which to client certificates will be copied to. Defaults to `~/ansible_persistent_files/docker_certificates`                      |

If you have specified a list of clients in `docker_client_hostnames`, the
certificate for each client will be stored locally on your Ansible controller in
the folder `docker_client_certificate_cache_directory`. You will then need to
copy these certificates to the corresponding client.

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: mirsg.infrastructure.docker
```
