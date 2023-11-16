# Role contents

Below overview shows the entire file and folder structure including their description.

## Structure

```
├── defaults                                        # The native defaults variables directory.
│   └── main.yml                                    # File containing a set of default variables.
│
├── docs                                            # The documentation directory.
│   ├── examples                                    # Directory containing examples.
│   │   ├── monitoring_server                       # Dummy role directory.
│   │   │   └── vars                                # Dummy vars directory.
│   │   │       └── main.yml                        # Dummy file containing the current defaults.
│   │   │
│   │   └── roleplay.monitoring_server.yml          # Dummy file containing an example of an ansible play for this role.
│   │
│   ├── instructions.md                             # Instructions on how to use this role.
│   └── role_contents.md                            # Describing the contents of this role.            
│
├── files                                           # The native files directory.
│   └── dashboards                                  # The dashboard files directory.
|
├── handlers                                        # The native handler directory.
│   └── main.yml                                    # The handlers task file.
│
├── tasks                                           # The native tasks directory.
│   ├── configure_subtasks.yml                      # Task file for configuring the Grafana plugins, API keys, datasources, notifications and dashboards.
│   ├── configure.yml                               # Task file for configuring Prometheus, Node Exporter and basic Grafana settings.
│   ├── install.yml                                 # Task file for installing Prometheus, Node Exporter and Grafana.
│   ├── main.yml                                    # File containing tasks to orchestrate between the task files.
│   ├── preflight_subtasks.yml                      # Task file for preflight advanced settings before installation and configuration.
│   └── preflight.yml                               # Task file for preflight basic settings before installation and configuration.
│
├── templates                                       # The native template directory.
│   ├── grafana.ini.j2                              # A template for the Grafana basic configuration.
│   ├── ldap.toml.j2                                # A template for Grafana LDAP integration configuration.
│   ├── node_exporter.service.j2                    # A template for the Node Exporter service file.
│   ├── prometheus.service.j2                       # A template for the Prometheus service file.
│   ├── prometheus.yml.j2                           # A template for the Prometheus configuration.
│   ├── tmpfiles.j2                                 # A template for creating temporary Grafana dashboard files.
│   └── yum.repo.j2                                 # A template for creating yum repository files.
│
├── vars                                            # The native vars directory.
│   └── main.yml                                    # The main vars file.
│
├── CHANGELOG.md                                    # The default CHANGELOG file in markdown format.
└── README.md                                       # The default README file in markdown format.
```
