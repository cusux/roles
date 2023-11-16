# Instructions

Please read through this file to understand the general setup.  

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Instructions](#instructions)
	- [Download and Install](#download-and-install)
	- [Introduction](#introduction)
	- [Inventory host variables](#inventory-host-variables)
	- [User-defined variables](#user-defined-variables)
      - [Proxy settings](#proxy-settings)
      - [Prometheus and Node Exporter versioning](#prometheus-and-node-exporter-versioning)
      - [Prometheus and Node Exporter service listeners](#prometheus-and-node-exporter-service-listeners)
      - [Other Prometheus and Node Exporter configuration items](#other-prometheus-and-node-exporter-configuration-items)
      - [Grafana versioning](#grafana-versioning)
      - [Grafana basic configuration](#grafana-basic-configuration)
      - [Grafana basic security configuration](#grafana-basic-security-configuration)
      - [Grafana database configuration](#grafana-database-configuration)
      - [Grafana user management configuration](#grafana-user-management-configuration)
      - [Grafana server configuration](#grafana-server-configuration)
      - [Grafana authentication configuration](#grafana-authentication-configuration)
      - [Grafana dashboard configuration](#grafana-dashboard-configuration)
      - [Grafana alert noticiation configuration](#grafana-alert-noticiation-configuration)
      - [Grafana datasources configuration](#grafana-datasources-configuration)
      - [Grafana API keys configuration](#grafana-api-keys-configuration)
      - [Grafana environmental configuration](#grafana-environmental-configuration)
	- [Default variables](#default-variables)
      - [Predefined proxy settings](#predefined-proxy-settings)
      - [Predefined target system archtecture](#predefined-target-system-archtecture)
      - [Predefined preflight configuration](#predefined-preflight-configuration)
      - [Predefined Prometheus configuration](#predefined-Prometheus-configuration)
      - [Predefined Node Exporter configuration](#predefined-Node-Exporter-configuration)
      - [Predefined Grafana configuration](#predefined-Grafana-configuration)

<!-- /TOC -->

## Download and Install

Download this role and place it inside the ansible roles directory.  
Make sure to use the correct name: **monitoring_server**

The setup should look something like this:

```
.
└── ansible_project_directory
    ├── group_vars
    ├── host_vars
    ├── hosts
    └── roles
        └── monitoring_server               <===== Place the role here
```

The contents of the role are described in: [Role contents](role_contents.md)

## Introduction

The combination of Prometheus, Node Exporter and Grafana can become very cumbersome and complex if it's not configured with a logical structure. Due to the complex configuration structure of Grafana it is hard to fully utilize it's potential. All logical values are combined as much as possible in variable objects that can be defined entirely by the user. In the below sections each required part will be described in detail.

This file can contain references to the official Prometheus documentation: [https://prometheus.io/docs/](https://prometheus.io/docs/)
This file can contain references to the official Node Exporter documentation: [https://prometheus.io/docs/guides/node-exporter/](https://prometheus.io/docs/guides/node-exporter/)
This file can contain references to the official Grafana documentation: [https://grafana.com/docs/](https://grafana.com/docs/)

## Inventory host variables

The monitoring_server role is made to be standalone in a sense that it does not need to gather all host based facts from remote nodes. It would be unnecessary to access each remote node to gather IP address and hostname values. Therefore it is required to set inventory hostgroups for each node. Also, the monitoring server should be monitoring itself, therefore the server host needs to be in both server and client hostgroups in the inventory.

Following is a working examplatory inventory with minimum needs for this Ansible role:

```yaml
---
nodes:
    children:
        hosts:
            monitoring_server.domain.local:
                ansible_host: 192.0.2.0
                ansible_user: username
        monitoring_servers:
            hosts:
                monitoring_server.domain.local:
        monitoring_clients:
            hosts:
                monitoring_server.domain.local:
```

All the default fallbacks are made so things don't just break. But please consider to take the time to understand how Prometheus, Node Exporter and Grafana work individually, as combination and how you would like to implement them using this role.

## User-defined variables

### Proxy settings

For complex environments it can occur that due to network segregation both the Ansible controller and the target systems have separate proxy servers for outbound access. Below variables can be set to meet proxy requirements.

```yaml
monitoring_server_local_proxy_http: 'http://proxy.local.domain:8080'           # 1
monitoring_server_local_proxy_https: 'http://proxy.local.domain:8080'          # 2
monitoring_server_prometheus_proxy_http: 'http://proxy.remote.domain:8080'     # 3
monitoring_server_prometheus_proxy_https: 'http://proxy.remote.domain:8080'    # 4
```

1. The variable for the local `http` proxy on the Ansible controller.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
2. The variable for the local `https` proxy on the Ansible controller.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
3. The variable for the remote `http` proxy on the target systems.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
4. The variable for the remote `https` proxy on the target systems.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).

### Prometheus and Node Exporter versioning

The versions for Prometheus and Node Exporter can be set by the user. It is advisable to always install and configure the latest version due to possible implications due bugs and possible security vulnerabilities.

```yaml
monitoring_server_prometheus_version: latest                                    # 1
monitoring_server_prometheus_node_exporter_version: latest                      # 2
```

1. This variable defines the Prometheus version.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
2. This variable defines the Node Exporter version.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Prometheus and Node Exporter service listeners

For each environment different network specifications are in place. The user must define several listener configuration items to get Prometheus up and running.

```yaml
monitoring_server_prometheus_target_url: localhost                              # 1
monitoring_server_prometheus_web_listen_address: 0.0.0.0                        # 2
monitoring_server_prometheus_web_listening_port: 9090                           # 3
monitoring_server_prometheus_web_external_url: ''                               # 4
monitoring_server_prometheus_fqdn: 'http://prometheus.local.domain'             # 5
monitoring_server_prometheus_node_exporter_web_listen_address: '0.0.0.0:9100'   # 6
```

1. The variable defining the target URL for Prometheus
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
2. The variable containing the web listening Prometheus IP address. On default `0.0.0.0` will suffice.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. The variable containing the web listening TCP port for Prometheus.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
4. This variable defines the external web URL for Prometheus.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
5. The Fully Qualified Domain Name for the Prometheus host.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
6. This variable defines the Node Exporter web listening address including port number.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Other Prometheus and Node Exporter configuration items

There are a few more configuration options the user can define for Prometheus and Node Exporter regarding data storage and metrics collection.

```yaml
monitoring_server_prometheus_storage_retention: "30d"                               # 1
monitoring_server_prometheus_storage_retention_size: 0                              # 2

monitoring_server_prometheus_node_exporter_enabled_collectors:                      # 3
  - systemd                                                                         # 4
  - textfile:                                                                       # 5
      directory: '{{ monitoring_server_prometheus_node_exporter_textfile_dir }}'    # 6
  - filesystem:                                                                     # 7
      ignored-mount-points: '^/(sys|proc|dev)($|/)'                                 # 8
      ignored-fs-types: '^(sys|proc|auto)fs$'                                       # 9

monitoring_server_prometheus_node_exporter_disabled_collectors: []                  # 10
```

1. This variable defines the Prometheus storage data retention period.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
2. This variable defines the Prometheus storage data retention size. **NOTE: This feature is experimental. Supported: KB, MB, GB, TB, PB. Default: Bytes. '0' == infinite**
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
3. This list object contains the Node Exporter enabled collectors
    - It is **required** to have this object defined.
    - List items should be a string value (*no spaces*) or a dictionary.
4. The list item for the `systemd` collector.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
5. The dictionary object for the `textfile` collector.
    - It is **required** to have this object defined.
    - Dictionary definitions should be a string value (*no spaces*).
6. The dictionary key for the `textfile` collector directory.
    - It is **required** to have this dictionary key defined.
    - List items should be a string value (*no spaces*).
7. The dictionary object for the `filesystem` collector.
    - It is **optional** to have this object defined.
    - Dictionary definitions should be a string value (*no spaces*).
8. The dictionary key to specify which mount-points to ignore in the `filesystem` collector.
    - It is **optional** to have this dictionary key defined.
    - List items should be a string value containing `regex` syntax.
9. The dictionary key to specify which filesystems to ignore in the `filesystem` collector.
    - It is **optional** to have this dictionary key defined.
    - List items should be a string value containing `regex` syntax.
10. This list object contains the Node Exporter disabled collectors
    - It is **optional** to have this object defined.
    - List items should be a string value (*no spaces*) or a dictionary.

### Grafana versioning

The Grafana version can be set by the user. It is advisable to always install and configure the latest version due to possible implications by bugs and possible security vulnerabilities. Also, running the latest version of Grafana will provide full featured functionality and flexibility.

```yaml
monitoring_server_grafana_version: latest
```

### Grafana basic configuration

The basic configuration needs for Grafana indicate the service daemon enabler, listening configuration and several data directories.

```yaml
monitoring_server_grafana_service_enabled: false                                # 1
monitoring_server_grafana_address: 0.0.0.0                                      # 2
monitoring_server_grafana_port: 3000                                            # 3
monitoring_server_grafana_logs_dir: '/var/log/grafana'                          # 4
monitoring_server_grafana_data_dir: '/var/lib/grafana'                          # 5
monitoring_server_grafana_dashboards_dir: 'dashboards'                          # 6
```

1. This variable determines wether or not to enable the Grafana service daemon on system startup.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
2. This variable defines the Grafana listening IP address.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. This variable defines the Grafana listening port.
    - It is **required** to have this object defined.
    - Use a 16 bit integer value (<=`65535`).
4. The directory for storing the Grafana logs.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
5. The directory for storing Grafana data.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
6. The directory name for storing Grafana dashboards.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Grafana basic security configuration

The basic Grafana security configuration consists of the initial user to log on with after installation and configuration.

```yaml
monitoring_server_grafana_security:                                             # 1
  admin_user: admin                                                             # 2
  admin_password: "Grafana!1234"                                                # 3
  disable_gravatar: true                                                        # 4
  secret_key: ""                                                                # 5
  login_remember_days: 7                                                        # 6
  cookie_username: grafana_user                                                 # 7
  cookie_remember_name: grafana_remember                                        # 8
  data_source_proxy_whitelist:                                                  # 9
```

1. This dictionary object contains the basic Grafana security options.
    - It is **required** to have this object defined.
    - Dictionary definitions should be a string value (no spaces).
2. The dictionary key for the initial administration username.
    - It is **required** to have this dictionary key defined.
    - Value should be a string value (*no spaces*).
3. The dictionary key for the initial administration password.
    - It is **required** to have this dictionary key defined.
    - Value should be a string value (*no spaces*).
    - This value can be stored in Ansible Vault but is not yet supported by this Ansible role at this moment.
4. The dictionary key for the Grafana gravatar usage.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.
5. The dictionary key for the Grafana secret key.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.
6. The dictionary key for the Grafana login expiry.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.
7. The dictionary key for the Grafana cookie username.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.
8. The dictionary key for the Grafana cookie remember name.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.
9. The dictionary key for the Grafana datasource whitelist.
    - It is **optional** to have this dictionary key defined.
    - Please refer to the official Grafana documentation for usage and further explanatory.

### Grafana database configuration

Grafana has to have a database connection in order to store information. This section will describe the smallest configuration for the database connection.  
Please refer to the official Grafana documentation for all options, usage and further explanatory.

```yaml
monitoring_server_grafana_database:
  type: sqlite3
```

## Grafana user management configuration

In order to enable user management, we will only use the bare minimum configuration at this moment in time.  
Please refer to the official Grafana documentation for usage and further explanatory.

```yaml
monitoring_server_grafana_welcome_email_on_sign_up: false                       # 1

monitoring_server_grafana_users:                                                # 2
  allow_sign_up: false                                                          # 3
```

1. The variable defining wether or not to send a welcome mail after users have signed up.
    - It is **required** to have this variable defined.
    - Use a boolean value (`true`/`false`).
2. The dictionary object for Grafana user configuration.
    - It is **required** to have this object defined.
3. Enable or disable the sign-up feature for new users by setting this dictionary item.
    - It is **required** to have this item defined.
    - Use a boolean value (`true`/`false`).

### Grafana server configuration

In order to get the Grafana server running properly, a few minimum configuration requirements need to be met.  
Please refer to the official Grafana documentation for usage and further explanatory.

```yaml
monitoring_server_grafana_server:                                               # 1
protocol: http                                                                  # 2
enforce_domain: false                                                           # 3
socket: ""                                                                      # 4
cert_key: ""                                                                    # 5
cert_file: ""                                                                   # 6
enable_gzip: false                                                              # 7
static_root_path: public                                                        # 8
router_logging: false                                                           # 9
```

1. This dictionary object contains the Grafana server configurations.
    - It is **required** to have this object defined.
2. The dictionary key for the Grafana server connection protocol.
    - It is **required** to have this object defined.
    - Use a string value of one of the following: `http`, `https`, `socket`.
3. The dictionary key for the Grafana DNS rebind attack prevention.
    - It is **required** to have this object defined.
    - Use a string value (`true`/`false`). If set to (default) `false`, no redirection takes place if the host header doesn't match the domain.
4. The dictionary key for the socket path in case `protocol: socket`.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
5. The dictionary key for the server certificate key path in case `protocol: https`.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
6. The dictionary key for the server certificate file path in case `protocol: https`.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
7. The dictionary key to specify the usage of gzip compression.
    - It is **optional** to have this object defined.
    - Use a string value (`true`/`false`).
8. The dictionary key for the Grafana server frontend file path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
9. The dictionary key for the loglevel of HTTP requests. If set to (default) `false`, only errors will be logged.
    - It is **required** to have this object defined.
    - Use a string value (`true`/`false`).

### Grafana authentication configuration

This section describes the authentication configuration for Grafana.
For this role only a specific minimum is required, therefore not all options are used and/or defined. Some empty object definitions are needed.

Please refer to the official Grafana documentation for usage and further explanatory of the following configuration objects:

```yaml
monitoring_server_grafana_auth: {}
monitoring_server_grafana_ldap: {}
monitoring_server_grafana_session: {}

monitoring_server_grafana_analytics:
  reporting_enabled: false

monitoring_server_grafana_smtp: {}

monitoring_server_grafana_alerting:
  execute_alerts: false

monitoring_server_grafana_log:
monitoring_server_grafana_metrics: {}
monitoring_server_grafana_tracing: {}

monitoring_server_grafana_snapshots:
  external_enabled: false

monitoring_server_grafana_image_storage: {}

monitoring_server_grafana_plugins: []
```

### Grafana dashboard configuration

The Grafana dashboards are dynamically set by this role, depening on the provisioning options.

```yaml
monitoring_server_grafana_dashboards:                                           # 1
  - dashboard_id: '10283'                                                       # 2
    title: 'Pometheus Host Metrics'                                             # 3
    revision_id: '1'                                                            # 4
    datasource: 'Prometheus'                                                    # 5
```

1. This list object contains the dashboard the user wishes to deploy.
    - It is **required** to have this object defined.
    - It is **optional** to fill the object with items.
2. The dictionary key to specify the dashboard ID from [https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards).
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
3. The dictionary key to specify the visible dashboard name.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
4. The dictionary key to specify the dashboard ID's revision from [https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards).
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
5. The dictionary key to specify the datasource the dashboard needs to use.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).

### Grafana alert noticiation configuration

It is possible to set-up Grafana with alert notifications.  
For this role it is not necessary and therefore undocumented.

Please refer to the official Grafana documentation for usage and further explanatory of the alert notifications configuration.

```yaml
monitoring_server_grafana_alert_notifications: []
```

### Grafana datasources configuration

In order for Grafana to display dashboards based on metrics, at least one datasource is needed. This section will describe the smallest configuration for datasources.  
Please refer to the official Grafana documentation for all options, usage and further explanatory.

```yaml
monitoring_server_grafana_datasources:                                                                        # 1
  apiVersion: 1                                                                                               # 2
  deleteDatasources: []                                                                                       # 3
  datasources:                                                                                                # 4
    - name: Prometheus                                                                                        # 5
      type: prometheus                                                                                        # 6
      access: direct                                                                                          # 7
      url: '{{ monitoring_server_prometheus_fqdn }}:{{ monitoring_server_prometheus_web_listening_port }}'    # 8
      isDefault: true                                                                                         # 9
      jsonData:                                                                                               # 10
        tlsAuth: false                                                                                        # 11
        tlsAuthWithCACert: false                                                                              # 12
        tlsSkipVerify: true                                                                                   # 13
```

1. This dictionary object contains the Grafana datasources configuration.
    - It is **required** to have this object defined.
2. The dictionary key to specify which API version to utilize.
    - It is **required** to have this object defined.
    - Use an integer value.
3. This object contains the datasources which should be deleted by the Ansible role.
    - It is **optional** to have this object defined.
    - Refer to the official Grafana documentation for usage and syntax.
4. This list object contains datasources to implement in Grafana.
    - It is **required** to have this object defined.
5. The dictionary key to specify the name of the datasource.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
6. The dictionary key to specify the datasource type.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
7. The dictionary key to specify the datasource access method.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*). Possible options are `proxy` and `direct`.
8. The dictionary key to specify datasource URL.
    - It is **required** to have this object defined.
    - Value is dynamically created based on variable `monitoring_server_prometheus_fqdn` and `monitoring_server_prometheus_web_listening_port`.
9. The dictionary key to specify wether or not the datasource should be the default datasource.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
10. This dictionary object contains the connection configuration of the datasource.
    - It is **required** to have this object defined.
11. The dictionary key to specify if TLS authentication must be used.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
12. The dictionary key to specify if TLS authentication must use a CA certificate.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
13. The dictionary key to specify if TLS authentication should skip verification.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).

### Grafana API keys configuration

The Grafana instance can be setup for API usage. To manage API rights, this role will setup 3 API-keys.  
This section contains both the API-keys configuration as well as the API-keys location configuration.

```yaml
monitoring_server_grafana_api_keys:                                                   # 1
  - name: admin                                                                       # 2
    role: Admin                                                                       # 3
  - name: viewer
    role: Viewer
  - name: editor
    role: Editor

monitoring_server_grafana_api_keys_dir: "{{ lookup('env', 'HOME') }}/grafana/keys"    # 4
```

1. This list object contains the API-keys for the Grafana instance.
    - It is **optional** to have this object defined.
2. The dictionary key to specify the API-key name.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
3. The dictionary key to specify the API-key role.
    - It is **optional** to have this object defined.
    - Use a string value (*no spaces*).
4. The variable defining the API-key storage location.
    - It is **optional** to have this variable defined.
    - Value is dynamically created based on environmental variable for the `HOME` folder and the user-specified suffix.

### Grafana environmental configuration

For Grafana environment specifics can be set.  
The minimum requirements for this role doesn't include any Grafana environment specific configuration and therefore the object is empty.

```yaml
monitoring_server_grafana_environment: {}
```

## Default variables

Each Ansible role has a set of default variables which should not be altered by the user. To have a better understanding of the inner workings of this role, the default variables are described in this section.

### Predefined proxy settings

For complex environments it can occur that due to network segregation both the Ansible controller and the target systems have separate proxy servers for outbound access. Below objects will combine the user-specified proxy settings.

```yaml
local_proxy_env:                                                                # 1
  http_proxy: '{{ monitoring_server_local_proxy_http }}'                        # 2
  https_proxy: '{{ monitoring_server_local_proxy_https }}'                      # 3

proxy_env:                                                                      # 4
  http_proxy: '{{ monitoring_server_prometheus_proxy_http }}'                   # 5
  https_proxy: '{{ monitoring_server_prometheus_proxy_https }}'                 # 6
```

1. The name of the proxy list object `local_proxy_env` for the Ansible controller.
    - It is **optional** to have the object defined.
    - This object will be filled by parameters defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
2. The variable for the `http` proxy for the Ansible controller.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_server_local_proxy_http` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
3. The variable for the `https` proxy for the Ansible controller
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_server_local_proxy_https` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
4. The name of the proxy list object `proxy_env` for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameters defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
5. The variable for the `http` proxy for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_server_prometheus_proxy_http` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
6. The variable for the `https` proxy for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_server_prometheus_proxy_https` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).

### Predefined target system archtecture

For Prometheus and Node Exporter, the role will take the target systems architecture into account. Therefore a translation object, which maps the system architecture to the package architecture naming, is needed for playbook efficiency. **At least one architecture must be present for this role to work**

```yaml
monitoring_server_prometheus_arch_map:                                          # 1
  i386: '386'                                                                   # 2
  x86_64: 'amd64'                                                               # 3
  aarch64: 'arm64'                                                              # 4
  armv7l: 'armv7'                                                               # 5
  armv6l: 'armv6'                                                               # 6
```

1. The name of the architecture mappings dictionary object `monitoring_server_prometheus_arch_map`.
    - It is **required** to have the object defined.
    - The naming of this object's item keys is predefined based on the system architecture naming convention.
    - The naming of this object's item values is predefined based on the naming of the packages.
2. The mapping for `i386` systems.
    - This item is **optional** to be defined, depending on the target architectures.
    - Use a string value (*no spaces*).
3. The mapping for `x86_64` systems.
    - This item is **optional** to be defined, depending on the target architectures.
    - Use a string value (*no spaces*).
4. The mapping for `aarch64` systems.
    - This item is **optional** to be defined, depending on the target architectures.
    - Use a string value (*no spaces*).
5. The mapping for `armv7l` systems.
    - This item is **optional** to be defined, depending on the target architectures.
    - Use a string value (*no spaces*).
6. The mapping for `armv6l` systems.
    - This item is **optional** to be defined, depending on the target architectures.
    - Use a string value (*no spaces*).

After the mapping object is filled, Ansible can map the architecture to the package definition. This is done with the following variable:

```yaml
monitoring_server_prometheus_arch:
  '{{ monitoring_server_prometheus_arch_map[ansible_architecture] | default(ansible_architecture) }}'
```

### Predefined preflight configuration

Before packages can be installed and configured some variables need to be set in order for the installation and configuration tasks to complete.

```yaml
monitoring_server_yum_repo_template: yum.repo.j2                                # 1
monitoring_server_yum_sslcacert_usage: true                                     # 2
monitoring_server_yum_sslcacert_path: '/etc/pki/tls/certs'                      # 3

monitoring_server_yum_repos:                                                    # 4
  - repo:                                                                       # 5
      name: grafana                                                             # 6
      state: present                                                            # 7
      baseurl: 'https://packages.grafana.com/oss/rpm'                           # 8
      enabled: 1                                                                # 9
      gpgcheck: 1                                                               # 10
      gpgkey: 'https://packages.grafana.com/gpg.key'                            # 11
      sslcacert: '{{ monitoring_server_yum_sslcacert_path }}/ca-bundle.crt'     # 12
      exclude: ""                                                               # 13

monitoring_server_selected_modules:                                             # 14
  - node_exporter                                                               # 15
  - prometheus                                                                  # 16

monitoring_server_prometheus_path: '/opt/prometheus'                            # 17
monitoring_server_prometheus_db_path: db                                        # 18
monitoring_server_prometheus_metrics_path: '/metrics'                           # 19
monitoring_server_prometheus_config_file: prometheus.yml                        # 20
monitoring_server_prometheus_config_template: prometheus.yml.j2                 # 21
```

1. The name of the template used to build the neede Yum repositie file(*s*).
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
2. The variable defining wether or not to use only secure (*SSL*) based repositories.
    - This item is **required** to be defined. For security purpose this object is defaulted to `true`.
    - Use a boolean value (`true`/`false`).
3. The path to the system default SSL-certificate store.
    - This item is **optional**, depending on the SSL-usage enforcement.
    - Use a string value (*no spaces*).
4. The list object for creation of Yum repository files.
    - It is **required** to have the object defined.
5. The dictionary object for each individual Yum repository.
    - It is **required** to have at least one dictionar object as list item.
6. The name of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
7. The state of the repository file for Ansible task interpretation.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `present` or `absent`.
8. The base url of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
9. The variable defining wether or not the repository is enabled.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `1` or `0`.
10. The variable defining wether or not to check the GPG key of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `1` or `0`.
11. The location of the GPG key for the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
12. The location of the SSL CA Certificate trusting the repository server.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) prepended by the variable `monitoring_server_yum_sslcacert_path` defining the SSL path.
13. This variable defines packages to exlcude during installation and updates.
    - It is **optional** to have the object defined.
    - Use a string value (*no spaces*).
    - Multiple string values are allowed by comma separation.
    - Shell globs using wildcards are allowed (`*`,`?`).
14. This list object defines which modules should be installed. By default Prometheus will monitor itself by means of the Node Exporter.
    - It is **required** to have this object defined.
    - List items **must** be added in order of installation.
15. This list item defines the `node_exporter` monitoring server module to install.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
16. This list item defines the `prometheus` monitoring server module to install.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
17. This variable defines the Prometheus installation path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
18. This variable defines the Prometheus database path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
19. This variable defines the Prometheus metric path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
20. This variable defines the Prometheus configuration filename.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
21. This variable defines the name of the Prometheus configuration file template.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Predefined Prometheus configuration

This role is configured to monitor the monitoring server itself. Therefore a basic set of scrape configurations is pre-defined.

```yaml
monitoring_server_prometheus_scrape_configs:                                               # 1
  - job_name: localhost_prometheus                                                         # 2
    metrics_path: '{{ monitoring_server_prometheus_metrics_path }}'                        # 3
    static_configs:                                                                        # 4
      - targets:                                                                           # 5
          - "{{ monitoring_server_prometheus_target_url | default('localhost') }}:9090"    # 6
  - job_name: localhost_node_exporter                                                      # 7
    static_configs:                                                                        # 8
      - targets:                                                                           # 9
          - "{{ monitoring_server_prometheus_target_url | default('localhost') }}:9100"    # 10
```

1. This list object containing the scrape configurations used by Prometheus.
    - It is **required** to have this object defined.
    - List items for self monitoring are predefined and thus **required**.
2. The dictionary object containing the initial job name for Prometheus.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. This variable defines the metrics path for the specific scrape job based on the predefined `monitoring_server_prometheus_metrics_path` variable.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
4. The list object containing the static scrape configuration.
    - It is **required** to have this object defined.
5. This list object contains the scrap targets.
    - It is **required** to have this object defined.
6. This list item contains the target URL for this scrape configuration job. This value defaults if none is specified.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
7. The dictionary object containing the initial job name for Node Exporter.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
8. The list object containing the static scrape configuration.
    - It is **required** to have this object defined.
9. This list object contains the scrap targets.
    - It is **required** to have this object defined.
10. This list item contains the target URL for this scrape configuration job. This value defaults if none is specified.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Predefined Node Exporter configuration

The Node Exporter needs to be configured for textfile metrics collection. This is done by the following variable:

```yaml
# Node Exporter textfile directory
monitoring_server_prometheus_node_exporter_textfile_dir:
  '{{ monitoring_server_prometheus_path }}/node_exporter'
```

### Predefined Grafana configuration

Due to the fact that Grafana is installed using a Yum repository, several configuration options need to be predefined. To further utilize Grafana => 5.0 some options have been made predefined as well to keep the user from accidentally editing them. Further information will be provided in the explanatory of each object.

To default the version and get Yum ready, some smart filtering with variable `monitoring_server_grafana_version` is needed to send the correct version to Yum:

```yaml
monitoring_server_grafana_package:
  "grafana{{ (monitoring_server_grafana_version != 'latest') | ternary('-' ~ monitoring_server_grafana_version, '') }}"
```

Further variables are defaulted and therefore they don't need to be defined by the user in most situations:

```yaml
monitoring_server_grafana_dependencies: []                                                                              # 1

monitoring_server_grafana_config_paths:                                                                                 # 2
  - '/etc/grafana'                                                                                                      # 3
  - '/etc/grafana/datasources'                                                                                          # 4
  - '/etc/grafana/provisioning'                                                                                         # 5
  - '/etc/grafana/provisioning/datasources'                                                                             # 6

monitoring_server_grafana_additional_paths:                                                                             # 7
  - '{{ monitoring_server_grafana_logs_dir }}'                                                                          # 8
  - '{{ monitoring_server_grafana_data_dir }}'                                                                          # 9
  - '{{ monitoring_server_grafana_data_dir }}/dashboards'                                                               # 10
  - '{{ monitoring_server_grafana_data_dir }}/plugins'                                                                  # 11

monitoring_server_grafana_url: 'http://{{ monitoring_server_grafana_address }}:{{ monitoring_server_grafana_port }}'    # 12
monitoring_server_grafana_api_url: '{{ monitoring_server_grafana_url }}'                                                # 13
monitoring_server_grafana_domain: '{{ ansible_fqdn | default(ansible_host) | default('localhost') }}'                   # 14
monitoring_server_grafana_use_provisioning: true                                                                        # 15
monitoring_server_grafana_provisioning_synced: true                                                                     # 16
monitoring_server_grafana_instance: '{{ ansible_fqdn | default(ansible_host) | default(inventory_hostname) }}'          # 17
```

1. This list object defines wether or not there are dependencies for Grafana installation.
    - It is **optional** to have this object defined.
    - List items should be a string value (*no spaces*).
2. This list object defines the paths for the basic Grafana configuration files.
    - It is **required** to have this object defined.
    - List items should be a string value (*no spaces*).
3. The parent Grafana configuration path.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
4. The configuration path for the Grafana datasources.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
5. The parent Grafana provisioning module configuration path.
    - It is **optional** to have this list item defined since it is only needed when the provisioning module is being used.
    - Use a string value (*no spaces*).
6. The configuration patg for the Grafana provisioning module datasources.
    - It is **optional** to have this list item defined since it is only needed when the provisioning module is being used.
    - Use a string value (*no spaces*).
7. This list object defines the paths for the Grafana log and storage configuration files.
    - It is **required** to have this object defined.
    - List items should be a string value (*no spaces*).
8. The Grafana log path, user-defined by the `monitoring_server_grafana_logs_dir` variable.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
9. The Grafana main data path, user-defined by the `monitoring_server_grafana_data_dir` variable.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
10. The Grafana data path for dashboards.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
11. The Grafana data path for plugins.
    - It is **required** to have this list item defined.
    - Use a string value (*no spaces*).
12. The public Grafana URL, compiled of the user-defined `monitoring_server_grafana_address` and `monitoring_server_grafana_port` variables
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
13. The public Grafana API URL, by default and for lower impact it is set to the `monitoring_server_grafana_url` variable.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
14. The Fully Qualified Domain Name of the Grafana host, based on the `ansible_fqdn` fact, defaulted to the `ansible_host` fact with fallback to `localhost`.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
15. Wether or not to use the Grafana provisioning module. **NOTE: This is only applicable for Grafana version => 5.0**
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
16. Wether or not to synchronize the Ansible controller to the Grafana server when using provisioning.
    - It is **required** to have this object defined.
    - Use a boolean value (`true`/`false`).
17. The instance name for the Grafana server, based on the `ansible_fqdn` fact, defaulted to the `ansible_host` fact with fallback to the `inventory_hostname` variable.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

To enable the use of ports below portnumber `1024` for unprivileged processes, Linux needs to set `CAP_NET_BIND_SERVICE` .  
This is defaulted to `false` due to the security implications. Enabling this option should not be taken lightly and should be a conscious decision.  
For more information, please refer to [http://man7.org/linux/man-pages/man7/capabilities.7.html](http://man7.org/linux/man-pages/man7/capabilities.7.html).

```yaml
monitoring_server_grafana_cap_net_bind_service: false
```
