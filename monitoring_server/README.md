# Monitoring server

## Description
Install and configure Prometheus with Node Exporter and Grafana as the "monitoring server".

## Guidelines
Prometheus and Node Exporter aren't rocket science.  
Complexity of the role is brought by Grafana and the coherence between all 3 components.

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
