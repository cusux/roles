# Role contents

Below overview shows the entire file and folder structure including their description.

## Structure

```
├── defaults                                        # The native defaults variables directory.
│   └── main.yml                                    # File containing a set of default variables.
│
├── docs                                            # The documentation directory.
│   ├── examples                                    # Directory containing examples.
│   │   ├── monitoring_client                       # Dummy role directory.
│   │   │   └── vars                                # Dummy vars directory.
│   │   │       └── main.yml                        # Dummy file containing the current defaults.
│   │   │
│   │   └── roleplay.monitoring_client.yml          # Dummy file containing an example of an ansible play for this role.
│   │
│   ├── instructions.md                             # Instructions on how to use this role.
│   └── role_contents.md                            # Describing the contents of this role.            
│
├── handlers                                        # The native handler directory.
│   └── main.yml                                    # The handlers task file.
│
├── tasks                                           # The native tasks directory.
│   ├── configure_subtasks.yml                      # Task file to update the monitoring server configuration.
│   ├── configure.yml                               # Task file for configuring Prometheus and InfluxData selected modules.
│   ├── install.yml                                 # Task file for installing Prometheus and InfluxData selected modules.
│   ├── main.yml                                    # File containing tasks to orchestrate between the task files.
│   ├── preflight_subtasks.yml                      # Task file for preflight advanced settings before installation and configuration.
│   └── preflight.yml                               # Task file for preflight basic settings before installation and configuration.
│
├── templates                                       # The native template directory.
│   ├── influxdb.conf.j2                            # A template for the InfluxData InfluxDB basic configuration.
│   ├── node_exporter.service.j2                    # A template for the Prometheus Node Exporter service file.
│   ├── telegraf.conf.j2                            # A template for the InfluxData Telegraf basic configuration.
│   └── yum.repo.j2                                 # A template for creating yum repository files.
│
├── vars                                            # The native vars directory.
│   └── main.yml                                    # The main vars file.
│
├── CHANGELOG.md                                    # The default CHANGELOG file in markdown format.
└── README.md                                       # The default README file in markdown format.
```
