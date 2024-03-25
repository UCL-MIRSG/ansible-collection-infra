# Ansible Role: mirsg.tomcat

A role for installing and configuring Apache Tomcat.

## Role Variables

### SELinux settings

`selinux_enabled`: Specify whether `selinux` is enabled on the host. Defaults to
`false`.

### Java settings

`java_home`: Path to java installation. Defaults to "/usr/lib/jvm/jre".

`java_profile_d`: Directory in which to put a script for setting java home. Defaults to "/etc/profile.d".

### Tomcat general settings

`tomcat_version`: The version of Tomcat to install. Defaults to the latest release of version 9.

`tomcat_owner`: The OS user that has ownership of Tomcat. Defaults to "tomcat".

`tomcat_group`: The default OS group the `tomcat_owner` belongs in. Defaults to "tomcat".

### Tomcat WebApp settings

`tomcat_webapp_name`: The name of the root web app. Defaults to "ROOT".

`tomcat_root`: The root web app location. Defaults to "/usr/share/tomcat/webapps/{{
tomcat_webapp_name }}".

`tomcat_root_webapp`: Path to the root web app war file. Defaults to
"{{ tomcat_root }}.war".

`tomcat_binary_url`: The URL of the binary to install from. Defaults to:

```yaml
"https://archive.apache.org/dist/tomcat/tomcat-\
{{ tomcat_version.split('.')[0] }}/v{{ tomcat_version }}/bin/\
apache-tomcat-{{ tomcat_version }}.tar.gz"
```

### Catalina settings

`tomcat_catalina_home`: The installation location. Defaults to "/usr/share/tomcat".

`tomcat_catalina_opts`: Sets to `CATALINA_OPTS` environment variable. Defaults to:

```yaml
"-Xms4G -Xmx6G -XX:MetaspaceSize=300M -XX:+UseG1GC -server"
```

### Tomcat config files

`tomcat_config_file`: The location of the configuration file. Defaults to
"/usr/share/tomcat/conf/tomcat.conf".

`tomcat_server_config_file`: The web app configuration file. Defaults to
"/usr/share/tomcat/conf/server.xml".

`tomcat_service_config_file`: The location of the systemd service file. Defaults to
"/etc/systemd/system/tomcat.service".

### Tomcat hostname and ports

`tomcat_hostname`: The hostname of the deployed web app. Defaults to `localhost`.

`tomcat_server_port`: The server port. Defaults to `8005`.

`tomcat_catalina_port`: The catalina port. Defaults to `8983`.

`tomcat_catalina_redirect_port`: Catalina port for redirects. Defaults to `8443`.

`tomcat_shutdown_port`: Port for triggering server shutdown. Defaults to `8005`.

`tomcat_port`: The web app HTTP port. Defaults to `8080`.

### Tomcat back settings

`tomcat_backup_directory`: Where to backup files to before an upgrade. Defaults to
`/usr/share/tomcat_bkp`.

`tomcat_items_to_restore`: A list containing the following items to be restored after
an upgrade. Defaults to:

```yaml
- "{{ tomcat_backup_directory }}/webapps"
- "{{ tomcat_backup_directory }}/logs"
```

## Example Playbook

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  roles:
    - role: mirsg.infrastructure.tomcat
```
