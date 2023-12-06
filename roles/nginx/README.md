# mirsg.nginx

Installs and configures an [nginx](https://www.nginx.com/) reverse proxy server.

## Requirements

Covered by [requirements.yml](https://github.com/UCL-MIRSG/ansible-collection-infra/blob/main/meta/requirements.yml).

## Role Variables

See defaults/main.yml for the full list.

- `owner`: Defaults to root
- `group`: Defaults to root
- `log_folder`: Defaults to "/var/log/nginx"
- `http_port`: http port to listen on: Defaults to 80
- `https_port`: https to redirect to: Defaults to 443
- `certs_dir`: Defaults to "/etc/nginx/ssl"
- `dh_params_file`: Location to store parameters for Diffie-Hellman key exchange, defaults to "/etc/nginx/ssl/dhparam.pem"
- conf_file: Location to store the nginx configuration to: Defaults to "/etc/nginx/nginx.conf", copied from `templates/nginx.j2`
- ssl_cert_file: Location to copy ssl.server_cert to: Defaults to "/etc/nginx/ssl/server.cert"
- ssl_key_file: Location to copy the ssl.server_key to: Defaults to "/etc/nginx/ssl/server.key"
- nginx_diffie_helman_size_bits:
  Bit size for OpenSSL Diffie-Hellman Parameters. Higher bit sizes are more
  secure, but require exponentially larger times for the one-off parameter
  generation. Use 4096 for production. These may take 10mins+ to generate but
  are only generated once per server.
  For local testing (non-production), use 2048 to speed up deployment:
  Defaults to 4096

## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

## License

BSD

## Author Information

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
