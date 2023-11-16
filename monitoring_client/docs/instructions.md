# Instructions

Please read through this file to understand the general setup.  

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Instructions](#instructions)
	- [Download and Install](#download-and-install)
	- [Introduction](#introduction)
	- [Inventory host variables](#inventory-host-variables)
	- [User-defined variables](#user-defined-variables)
      - [Proxy settings](#proxy-settings)
      - [Prometheus and InfluxData modules versioning](#prometheus-and-influxdata-modules-versioning)
      - [Prometheus Node Exporter service listeners](#prometheus-node-exporter-service-listeners)
      - [Other Prometheus Node Exporter configuration items](#other-prometheus-node-exporter-configuration-items)
      - [InfluxData Telegraf agent configuration](#influxdata-telegraf-agent-configuration)
      - [InfluxData Telegraf output plugin configuration](#influxdata-telegraf-output-plugin-configuration)
      - [InfluxData Telegraf input plugin configuration](#influxdata-telegraf-input-plugin-configuration)
      - [InfluxData Telegraf service input plugin configuration](#influxdata-telegraf-service-input-plugin-configuration)
      - [InfluxData InfluxDB metastore configuration](#influxdata-influxdb-metastore-configuration)
	- [Default variables](#default-variables)
      - [Predefined proxy settings](#predefined-proxy-settings)
      - [Predefined target system archtecture](#predefined-target-system-archtecture)
      - [Predefined preflight configuration](#predefined-preflight-configuration)
      - [Predefined Prometheus configuration](#predefined-prometheus-configuration)
      - [Predefined Node Exporter configuration](#predefined-node-exporter-configuration)

<!-- /TOC -->

## Download and Install

Download this role and place it inside the ansible roles directory.  
Make sure to use the correct name: **monitoring_client**

The setup should look something like this:

```
.
└── ansible_project_directory
    ├── group_vars
    ├── host_vars
    ├── hosts
    └── roles
        └── monitoring_client               <===== Place the role here
```

The contents of the role are described in: [Role contents](role_contents.md)

## Introduction

The combination of Prometheus Node Exporter, InfluxData InfluxDB and InfluxData Telegraf can become very cumbersome and complex if it's not configured with a logical structure. Due to the complex configuration structure of the InfluData selected modules it is hard to fully utilize it's potential. All logical values are combined as much as possible in variable objects that can be defined entirely by the user. In the below sections each required part will be described in detail.

This file can contain references to the official Prometheus Node Exporter documentation: [https://prometheus.io/docs/guides/node-exporter/](https://prometheus.io/docs/guides/node-exporter/)  
This file can contain references to the official InfluxData InfluxDB documentation: [https://docs.influxdata.com/influxdb/](https://docs.influxdata.com/influxdb/)  
This file can contain references to the official InfluxData Telegraf documentation: [https://docs.influxdata.com/telegraf/](https://docs.influxdata.com/telegraf/)  

## Inventory host variables

The monitoring_client role is made to be standalone in a sense that it does not need to gather all host based facts from remote nodes. It would be unnecessary to access each remote node to gather IP address and hostname values. Therefore it is required to set inventory hostgroups for each node.

Following is a working examplatory inventory with minimum needs for this Ansible role:

```yaml
---
nodes:
    children:
        hosts:
            monitoring_server.domain.local:
                ansible_host: 192.0.2.10
                ansible_user: username
            monitoring_client.domain.local:
                ansible_host: 192.0.2.11
                ansible_user: username
        monitoring_servers:
            hosts:
                monitoring_server.domain.local:
        monitoring_clients:
            hosts:
                monitoring_client.domain.local:
```

All the default fallbacks are made so things don't just break. But please consider to take the time to understand how Prometheus Node Exporter, InfluxData InfluxDB and InfluxData Telegraf work individually, as combination and how you would like to implement them using this role.

## User-defined variables

### Proxy settings

For complex environments it can occur that due to network segregation both the Ansible controller and the target systems have separate proxy servers for outbound access. Below variables can be set to meet proxy requirements.

```yaml
monitoring_server_local_proxy_http: 'http://proxy.local.domain:8080'            # 1
monitoring_server_local_proxy_https: 'http://proxy.local.domain:8080'           # 2
monitoring_server_prometheus_proxy_http: 'http://proxy.remote.domain:8080'      # 3
monitoring_server_prometheus_proxy_https: 'http://proxy.remote.domain:8080'     # 4
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

### Prometheus and InfluxData modules versioning

The versions for the selected Prometheus and InfluxData modules can be set by the user. It is advisable to always install and configure the latest version due to possible implications due bugs and possible security vulnerabilities.

```yaml
monitoring_client_prometheus_node_exporter_version: latest                      # 1
monitoring_client_influxdata_influxdb_version: latest                           # 2
monitoring_client_influxdata_telegraf_version: latest                           # 3
```

1. This variable defines the Prometheus Node Exporter version.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
2. This variable defines the InfluxData InfluxDB version.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. This variable defines the InfluxData Telegraf version.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Prometheus Node Exporter service listeners

For each environment different network specifications are in place. The user must define several listener configuration items to get Prometheus up and running.

```yaml
monitoring_client_prometheus_node_exporter_web_listen_address: '0.0.0.0:9100'   # 1
```

1. This variable defines the Node Exporter web listening address including port number.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Other Prometheus Node Exporter configuration items

There are a few more configuration options the user can define for Prometheus and Node Exporter regarding data storage and metrics collection.

```yaml
monitoring_server_prometheus_node_exporter_enabled_collectors:                      # 1
  - systemd                                                                         # 2
  - textfile:                                                                       # 3
      directory: '{{ monitoring_server_prometheus_node_exporter_textfile_dir }}'    # 4
  - filesystem:                                                                     # 5
      ignored-mount-points: '^/(sys|proc|dev)($|/)'                                 # 6
      ignored-fs-types: '^(sys|proc|auto)fs$'                                       # 7

monitoring_server_prometheus_node_exporter_disabled_collectors: []                  # 8
```

1. This list object contains the Node Exporter enabled collectors
    - It is **required** to have this object defined.
    - List items should be a string value (*no spaces*) or a dictionary.
2. The list item for the `systemd` collector.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. The dictionary object for the `textfile` collector.
    - It is **required** to have this object defined.
    - Dictionary definitions should be a string value (*no spaces*).
4. The dictionary key for the `textfile` collector directory.
    - It is **required** to have this dictionary key defined.
    - List items should be a string value (*no spaces*).
5. The dictionary object for the `filesystem` collector.
    - It is **optional** to have this object defined.
    - Dictionary definitions should be a string value (*no spaces*).
6. The dictionary key to specify which mount-points to ignore in the `filesystem` collector.
    - It is **optional** to have this dictionary key defined.
    - List items should be a string value containing `regex` syntax.
7. The dictionary key to specify which filesystems to ignore in the `filesystem` collector.
    - It is **optional** to have this dictionary key defined.
    - List items should be a string value containing `regex` syntax.
8. This list object contains the Node Exporter disabled collectors
    - It is **optional** to have this object defined.
    - List items should be a string value (*no spaces*) or a dictionary.

After installation and configuration, the Ansible role will update the Prometheus monitoring server.  
Below variable will define the hostname of the Prometheus monitoring server in order to be updated.

```yaml
monitoring_client_prometheus_server_fqdn: prometheus_server.domain.local
```

### InfluxData Telegraf agent configuration

The InfluxData Telegraf agent needs to be configured with user-defined settings for metrics collection and logging.  
Whilst below configuration settings appear to be complete, it is not.  
For a full list of configurable options and more information about each configurable option, refer to the official InfluxData Telegraf documentation: [https://docs.influxdata.com/telegraf/](https://docs.influxdata.com/telegraf/).

```yaml
monitoring_client_influxdata_telegraf_agent_configuration:                      # 1
  round_interval: 'true'                                                        # 2
  metric_batch_size: 1000                                                       # 3
  metric_buffer_limit: 10000                                                    # 4
  collection_jitter: '"0s"'                                                     # 5
  flush_interval: '"10s"'                                                       # 6
  flush_jitter: '"0s"'                                                          # 7
  precision: '""'                                                               # 8
  debug: 'false'                                                                # 9
  quiet: 'false'                                                                # 10
  logfile: '""'                                                                 # 11
  logfile_rotation_interval: '"0d"'                                             # 12
  logfile_rotation_max_size: '"0MB"'                                            # 13
  logfile_rotation_max_archives: 5                                              # 14
  hostname: '""'                                                                # 15
  omit_hostname: 'false'                                                        # 16
```

1. This dictionary object contains the InfluxData Telegraf agent configuration
    - It is **required** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
2. The dictionary key to specify wether or not to round the collection interval to the preset interval.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case, double quoted string value (*no spaces*) of either `true` of `false`.
		- String notation of Boolean value is necessary due to templating and conversion.
3. The dictionary key to specify the maximum batch size for metrics output.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a integer value (*no spaces*).
4. The dictionary key to specify the max buffer size for the batch output.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a integer value (*no spaces*).
		- This value should be a `multiple` of `metric_batch_size` and no less than 2 times `metric_batch_size`.
5. The dictionary key to specify the `jitter` for metrics order collection.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*) with the user-defined jitter interval.
		- Quotation is necessary due to templating and conversion.
6. The dictionary key to specify data flushing interval.
    - It is **required** to have this dictionary key defined.
		- Dictionary value should be a lower case single and double quoted string value (*no spaces*) with the user-defined flush interval.
		- Quotation is necessary due to templating and conversion.
7. The dictionary key to specify the `jitter` for flush interval during collection.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*) with the user-defined jitter interval.
		- Quotation is necessary due to templating and conversion.
8. The dictionary key to specify the collection interval precision.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*).
		- Valid values are `ns`, `us` (or `µs`), `ms`, and `s`.
		- Quotation is necessary due to templating and conversion.
9. The dictionary key to specify wether or not to enable debug information.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case, double quoted string value (*no spaces*) of either `true` of `false`.
		- String notation of Boolean value is necessary due to templating and conversion.
10. The dictionary key to specify wether or not to silence the output to mere error messages only.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case, double quoted string value (*no spaces*) of either `true` of `false`.
		- String notation of Boolean value is necessary due to templating and conversion.
11. The dictionary key to specify a custom logfile location.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*).
		- Quotation is necessary due to templating and conversion.
12. The dictionary key to specify the log rotation time interval.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*).
		- Quotation is necessary due to templating and conversion.
13. The dictionary key to specify the log rotation filesize interval.
    - It is **optional** to have this dictionary key defined.
		- Dictionary value should be a lower case single and double quoted string value (*no spaces*).
		- Quotation is necessary due to templating and conversion.
14. The dictionary key to specify the number of rotated log archives to retain.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a integer value (*no spaces*).
15. The dictionary key to override the default `os.Hostname()` in the metrics collection.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case single and double quoted string value (*no spaces*).
		- Quotation is necessary due to templating and conversion.
16. The dictionary key to specify wether or not to set the `host` tag.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case, double quoted string value (*no spaces*) of either `true` of `false`.
		- String notation is necessary due to templating and conversion.

### InfluxData Telegraf output plugin configuration

InfluxData Telegraf needs to be configured with user-defined settings for metrics storage and availability. This is done via the `output plugin`.  
For a full list of configurable options and more information about each configurable option, refer to the official InfluxData Telegraf documentation: [https://docs.influxdata.com/telegraf/](https://docs.influxdata.com/telegraf/).

```yaml
monitoring_client_influxdata_telegraf_output_plugins:                           # 1
  influxdb:                                                                     # 2
    urls: '["http://localhost:8086"]'                                           # 3
    database: '"telegraf"'                                                      # 4
    retention_policy: '""'                                                      # 5
    write_consistency: '"any"'                                                  # 6
    timeout: '"5s"'                                                             # 7
```

1. This dictionary object contains the InfluxData Telegraf output plugin configuration
    - It is **required** to have this object defined.
    - Dictionary items should be a string value (*no spaces*).
2. This dictionary item contains the InfluxData Telegraf output plugin configuration for the `influxdb` output plugin.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
3. The dictionary key which specifies the `url` for the `influxdb` instance.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case single quoted string value (*no spaces*).
		- Quotation is necessary due to templating and conversion.
4. The dictionary key which specifies the `database` to use for the `influxdb` instance.
    - It is **required** to have this dictionary key defined.
    - Dictionary value should be a lower case single quoted string value (*no spaces*).
    - Quotation is necessary due to templating and conversion.
5. The dictionary key which specifies the `retention policy` to write to on the `influxdb` instance.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case, double and single quoted string value (*no spaces*).
    - Quotation is necessary due to templating and conversion.
6. The dictionary key which specifies the `write consistency` for the clustered `influxdb` instance.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case, double and single quoted string value (*no spaces*).
    - Quotation is necessary due to templating and conversion.
7. The dictionary key which specifies the `HTTP message timeout` for the `influxdb` instance.
    - It is **optional** to have this dictionary key defined.
    - Dictionary value should be a lower case, double and single quoted string value (*no spaces*).
    - Quotation is necessary due to templating and conversion.

### InfluxData Telegraf input plugin configuration

To be able to collect metric, InfluxData Telegraf needs to be configured with `input plugins`.  
For a full list of configurable options per plugin and more information about each configurable option, refer to the official InfluxData Telegraf documentation: [https://docs.influxdata.com/telegraf/](https://docs.influxdata.com/telegraf/).  
This section will only describe the InfluxData Telegraf native measurement filtering options. These options apply to each input, output, processor or aggregator configured.

```yaml
monitoring_client_influxdata_telegraf_input_plugins:                            # 1
  plugingid:                                                                    # 2
    namepass: '["emit_pattern"]'                                                # 3
    namedrop: '["discard_patterd"]'                                             # 4
    fieldpass: '["field1","field3","field5"]'                                   # 5
    fielddrop: '["field2","field4","field6"]'                                   # 6
    tagpass:                                                                    # 7
      tag1: [ "option1", "option2" ]                                            # 8
    tagdrop:                                                                    # 9
      tag2: [ "option1", "option2" ]                                            # 10
    taginclude: '["emit_pattern"]'                                              # 11
    tagexclude: '["discard_pattern"]'                                           # 12
```

1. This dictionary object contains the InfluxData Telegraf input plugin configuration
    - It is **required** to have this object defined.
    - Dictionary items should be a string value (*no spaces*).
2. This dictionary item contains a InfluxData Telegraf input plugin configuration set.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
3. This dictionary key is the filter which emits `names` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
4. This dictionary key is the filter which discards `names` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
5. This dictionary key is the filter which emits `fields` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
6. This dictionary key is the filter which discards `fields` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
7. This dictionary item contains a configuration `set of tags` to emit to the input collection.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
8. This dictionary key is the filter for `tags` to matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
9. This dictionary item contains a configuration `set of tags` to discard in the input collection.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
10. This dictionary key is the filter for `tags` to matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary value should be a string value (*no spaces*).
11. This dictionary key is the filter which emits `tag keys` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
12. This dictionary key is the filter which discards `tag keys` matching the value pattern.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).

### InfluxData Telegraf service input plugin configuration

To be able to collect metric, InfluxData Telegraf needs to be configured with `service input plugins`.  
For a full list of configurable options per plugin and more information about each configurable option, refer to the official InfluxData Telegraf documentation: [https://docs.influxdata.com/telegraf/](https://docs.influxdata.com/telegraf/).  
This section will only describe the InfluxData Telegraf native measurement filtering options. These options apply to each input, output, processor or aggregator configured.

```yaml
monitoring_client_influxdata_telegraf_service_input_plugins:                    # 1
```

1. This dictionary object contains the InfluxData Telegraf service input plugin configuration
    - It is **required** to have this object defined.
    - Dictionary items should be a string value (*no spaces*).

All configurable dictionary items, keys and values withing this dictionary are the same as mentioned in section [InfluxData Telegraf input plugin configuration](#influxdata-telegraf-input-plugin-configuration).

### InfluxData InfluxDB metastore configuration

InfluxData InfluxDB stores metadata storage on users, databases, retention policies, shards, and continuous queries.  
For a full list of configurable options and more information about metadata storage, refer to the official InfluxData InfluxDB documentation: [https://docs.influxdata.com/influxdb/](https://docs.influxdata.com/influxdb/).

```yaml
monitoring_client_influxdata_influxdb_metastore_config:                         # 1
  dir: '"/var/lib/influxdb/meta"'                                               # 2
```

1. This dictionary object contains the InfluxData InfluxDB metadata storage configuration
    - It is **required** to have this object defined.
    - Dictionary items should be a string value (*no spaces*).
2. This dictionary key is defines the `directory` where the metadata needs to be stored.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).

### InfluxData InfluxDB metastore configuration

InfluxData InfluxDB datastore settings control the actual shard data location and how it is flushed from the Write-Ahead Log (*WAL*).  
For a full list of configurable options and more information about metadata storage, refer to the official InfluxData InfluxDB documentation: [https://docs.influxdata.com/influxdb/](https://docs.influxdata.com/influxdb/).

```yaml
monitoring_client_influxdata_influxdb_datastore_config:                         # 1
  dir: '"/var/lib/influxdb/data"'                                               # 2
  wal-dir: '"/var/lib/influxdb/wal"'                                            # 3
  series-id-set-cache-size: 100                                                 # 4
```

1. This dictionary object contains the InfluxData InfluxDB data storage configuration
    - It is **required** to have this object defined.
    - Dictionary items should be a string value (*no spaces*).
2. This dictionary key is defines the directory where the `TSM engine` stores the TSM data.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
3. This dictionary key is defines the directory where the `Write-Ahead Log` files are stored.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).
4. This dictionary key is defines the threshold, in bytes, when an index write-ahead log (*WAL*) file will compact into an index file.
    - It is **optional** to have this object defined.
    - Dictionary keys should be a string value (*no spaces*).

## Default variables

Each Ansible role has a set of default variables which should not be altered by the user. To have a better understanding of the inner workings of this role, the default variables are described in this section.

### Predefined proxy settings

For complex environments it can occur that due to network segregation both the Ansible controller and the target systems have separate proxy servers for outbound access. Below objects will combine the user-specified proxy settings.

```yaml
local_proxy_env:                                                                # 1
  http_proxy: '{{ monitoring_client_local_proxy_http }}'                        # 2
  https_proxy: '{{ monitoring_client_local_proxy_https }}'                      # 3

proxy_env:                                                                      # 4
  http_proxy: '{{ monitoring_client_proxy_http }}'                              # 5
  https_proxy: '{{ monitoring_client_proxy_https }}'                            # 6
```

1. The name of the proxy list object `local_proxy_env` for the Ansible controller.
    - It is **optional** to have the object defined.
    - This object will be filled by parameters defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
2. The variable for the `http` proxy for the Ansible controller.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_client_local_proxy_http` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
3. The variable for the `https` proxy for the Ansible controller
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_client_local_proxy_https` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
4. The name of the proxy list object `proxy_env` for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameters defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
5. The variable for the `http` proxy for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_client_proxy_http` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).
6. The variable for the `https` proxy for the remote system.
    - It is **optional** to have the object defined.
    - This object will be filled by parameter `monitoring_client_proxy_https` as defined in the `/vars/main.yml` file.
    - Use a string value (*no spaces*).

### Predefined target system archtecture

For the Prometheus Node Exporter, the role will take the target systems architecture into account. Therefore a translation object, which maps the system architecture to the package architecture naming, is needed for playbook efficiency. **At least one architecture must be present for this role to work**

```yaml
monitoring_client_prometheus_arch_map:                                          # 1
  i386: '386'                                                                   # 2
  x86_64: 'amd64'                                                               # 3
  aarch64: 'arm64'                                                              # 4
  armv7l: 'armv7'                                                               # 5
  armv6l: 'armv6'                                                               # 6
```

1. The name of the architecture mappings dictionary object `monitoring_client_prometheus_arch_map`.
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
monitoring_client_prometheus_arch:
  '{{ monitoring_client_prometheus_arch_map[ansible_architecture] | default(ansible_architecture) }}'
```

### Predefined preflight configuration

Before packages can be installed and configured some variables need to be set in order for the installation and configuration tasks to complete.

```yaml
monitoring_client_yum_repo_template: yum.repo.j2                                # 1

monitoring_client_yum_repos:                                                    # 2
  - repo:                                                                       # 3
      name: influxdata                                                          # 4
      state: present                                                            # 5
      baseurl: 'https://repos.influxdata.com/rhel/$releasever/$basearch/stable' # 6
      enabled: 1                                                                # 7
      gpgcheck: 1                                                               # 8
      gpgkey: 'https://repos.influxdata.com/influxdb.key'                       # 9
      exclude: ""                                                               # 10

monitoring_client_prometheus_selected_modules:                                  # 11
  - node_exporter                                                               # 12

monitoring_client_influxdata_selected_modules:                                  # 13
  - influxdb                                                                    # 14
  - telegraf                                                                    # 15

monitoring_client_prometheus_path: '/opt/prometheus'                            # 16
monitoring_client_prometheus_server_config_path:
  '{{ monitoring_client_prometheus_path }}'                                     # 17
monitoring_client_influxdata_influxdb_path: '/etc/influxdb'                     # 18
monitoring_client_influxdata_telegraf_path: '/etc/telegraf'                     # 19

monitoring_client_prometheus_server_config_file: prometheus.yml                 # 20
monitoring_client_influxdata_influxdb_config_file: influxdb.conf                # 21
monitoring_client_influxdata_telegraf_config_file: telegraf.conf                # 22

monitoring_client_influxdata_influxdb_config_template: influxdb.conf.j2         # 23
monitoring_client_influxdata_telegraf_config_template: telegraf.conf.j2         # 24
```

1. The name of the template used to build the neede Yum repositie file(*s*).
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
2. The list object for creation of Yum repository files.
    - It is **required** to have the object defined.
3. The dictionary object for each individual Yum repository.
    - It is **required** to have at least one dictionar object as list item.
4. The name of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
5. The state of the repository file for Ansible task interpretation.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `present` or `absent`.
6. The base url of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
7. The variable defining wether or not the repository is enabled.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `1` or `0`.
8. The variable defining wether or not to check the GPG key of the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*) of either `1` or `0`.
9. The location of the GPG key for the repository.
    - It is **required** to have the object defined.
    - Use a string value (*no spaces*).
10. This variable defines packages to exlcude during installation and updates.
    - It is **optional** to have the object defined.
    - Use a string value (*no spaces*).
    - Multiple string values are allowed by comma separation.
    - Shell globs using wildcards are allowed (`*`,`?`).
11. This list object defines which Prometheus modules should be installed. By default this will only contain the `Node Exporter` module but is expandable.
    - It is **required** to have this object defined.
    - List items **must** be added in order of installation.
12. This list item defines the `node_exporter` monitoring client module to install.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
13. This list object defines which InfluxData modules should be installed. By default this will only contain the `InfluxDB` and `Telegraf` modules but is expandable.
    - It is **required** to have this object defined.
    - List items **must** be added in order of installation.
14. This list item defines the `influxdb` monitoring client module to install.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
15. This list item defines the `telegraf` monitoring client module to install.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
16. This variable defines the `Prometheus modules` installation path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
17. This variable defines the `Prometheus modules` installation path.
    - It is **required** to have this object defined.
    - This variable will be automatically filled using the variable `monitoring_client_prometheus_path`.
18. This variable defines the `InfluxData InfluxDB` configuration path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
19. This variable defines the `InfluxData Telegraf` configuration path.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
20. This variable defines the `Prometheus server` configuration filename.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
21. This variable defines the `InfluxData InfluxDB` configuration filename.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
22. This variable defines the `InfluxData Telegraf` configuration filename.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
23. This variable defines the name of the `InfluxData InfluxDB` configuration file template.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
24. This variable defines the name of the `InfluxData Telegraf` configuration file template.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Predefined Prometheus configuration

This role is configured to monitor the specified clients. Therefore a basic scrape configuration is pre-defined to ensure monitoring of each client.

```yaml
monitoring_client_prometheus_scrape_configs:                                    # 1
  - job_name: '{{ inventory_hostname }}_node_exporter'                          # 2
    static_configs:                                                             # 3
      - targets:                                                                # 4
          - '{{ inventory_hostname }}:9100'                                     # 5
```

1. This list object containing the scrape configuration used by Prometheus monitoring server.
    - It is **required** to have this object defined.
    - List items for self monitoring are predefined and thus **required**.
2. The dictionary object containing the initial job name for Node Exporter.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).
3. The list object containing the static scrape configuration.
    - It is **required** to have this object defined.
4. This list object contains the scrape targets.
    - It is **required** to have this object defined.
5. This list item contains the target URL for this scrape configuration job.
    - It is **required** to have this object defined.
    - Use a string value (*no spaces*).

### Predefined Node Exporter configuration

The Prometheus Node Exporter module needs to be configured for textfile metrics collection. This is done by the following variable:

```yaml
monitoring_client_prometheus_node_exporter_textfile_dir: '{{ monitoring_client_prometheus_configpath }}/node_exporter'
```
