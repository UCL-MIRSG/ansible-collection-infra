# Ansible Role: mirsg.infrastructure.ca_trust

A role to manage the system-wide CA trust store (`/etc/pki/ca-trust/source`) on
EL 7, 9 and 10 hosts.

It distrusts the root CAs listed in the
[OpenJDK eMudhra advisory](https://openjdk.org/emudhra) (JDK-8395690). It can
also trust extra CA certificates, such as an LDAP server's CA.

Java reads the same trust store: the Temurin RPMs installed by
`mirsg.infrastructure.install_java` ship the JDK's `cacerts` as a link to
`/etc/pki/java/cacerts`. Never add certificates to that keystore directly with
`keytool` or `community.general.java_cert`, because it is regenerated whenever
the trust store is updated. Use `ca_trust_anchors` instead.

Each certificate in `ca_trust_anchors` is written to
`/etc/pki/ca-trust/source/<name>.p11-kit` and marked as a trusted authority.
`update-ca-trust` only exports certificates in `anchors/` that have `CA:TRUE`,
so a self-signed server certificate copied there would never reach Java's
`cacerts`.

When the trust store changes, the role sets the fact `ca_trust_changed: true`
for the rest of the run. Callers can use this to restart services that run on
the JVM.

## Role Variables

| Variable                 | Description                                                                                                                 | Default Value                    |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| `ca_trust_blocklist`     | Certificates in `files/blocklist/` to distrust.                                                                             | The eMudhra and Comodo AAA roots |
| `ca_trust_anchors`       | Extra certificates to trust: a list of `name`, `src` (a PEM file) and optional `remote_src` (true if `src` is on the host). | `[]`                             |
| `ca_trust_source_dir`    | The trust store source directory.                                                                                           | `/etc/pki/ca-trust/source`       |
| `ca_trust_extracted_dir` | The directory `update-ca-trust` extracts the trust store to.                                                                | `/etc/pki/ca-trust/extracted`    |

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: mirsg.infrastructure.ca_trust
      vars:
        ca_trust_anchors:
          - name: ldap-ca
            src:
              "{{ lookup('env', 'HOME') }}/ansible_persistent_files/ldap/ca.pem"
```
