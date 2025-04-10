# Ansible Role: mirsg.infrastructure.ldap

This Ansible role copies an LDAP cert from the controller to a remote machine
and adds it to the Java keystore. Make sure to install Java before running this
role.

## Role Variables

The following variables can be customized to suit your environment. Default
values are sourced from the `defaults` folder.

| Variable                   | Description                                                        | Default Value                            |
| -------------------------- | ------------------------------------------------------------------ | ---------------------------------------- |
| `ldap_cert_dir`            | Directory on the remote where the cert will be copied.             | `/root/certs`                            |
| `ldap_cert_owner`          | OS user on the remote that will have ownership of the copied cert. | `root`                                   |
| `ldap_cert_owner_group`    | OS group on the remote.                                            | `root`                                   |
| `ldap_ca_cert_path`        | Path to the cert on the Ansible controller.                        | N/A (must be provided by the user)       |
| `ldap_java_keystore_path`  | Path in the remote keystore to the cert.                           | `/usr/lib/jvm/jre/lib/security/cacerts/` |
| `ldap_java_keystore_pass`  | Password for the remote keystore.                                  | N/A (must be provided by the user)       |
| `ldap_java_keystore_alias` | Name of the cert in the remote keystore.                           | `ldap-ca`                                |

## Usage

To use this role, include it in your playbook as follows:

```yaml
- name: LDAP
  hosts: all
  roles:
    - role: mirsg.infrastructure.ldap
      vars:
        ldap_ca_cert_path:
          "{{ lookup('env', 'HOME') }}/ansible_persistent_files/ldap/ca.pem"
        ldap_java_keystore_path: /usr/lib/jvm/jre/lib/security/cacerts/
        ldap_java_keystore_pass: keystore_password
        ldap_java_keystore_alias: ldap-ca
```
