# Monitoring client

## Description
Install and configure Prometheus Node Exporter and InfluxData InfluxDB and Telegraf as the "monitoring client".

## Guidelines
The basic principles of Node Exporter, InfluxDB and Telegraf aren't rocket science.  
Complexity of the role is brought by the configuration of the InfluxData selected modules and the coherence between all components.

All user-defined variables are to be configured in `defaults/main.yml`.
All variables in `vars/main.yml` should not be adjusted by the user, unless they know what they're doing.

All configurable variables are sectioned, as described in the documentation in `docs/instructiond.md`.

## Installation and Usage
Before you start using this role, please read the [instructions](docs/instructions.md).

### Requirements & dependencies
- Requires outbound internet access, either through proxy or without proxy.
- Facts gathering is required.
- Works for Ansible version 2.8 and greater.

## Changelog
For the actual version and change history please read the [changelog](CHANGELOG.md).
