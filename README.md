# Ansible Datasaker Role

The Ansible Datasaker role installs and configures the Datasaker Agent and integrations.

## Setup

### Requirements

- Requires Ansible v2.6+.
- Supports most Debian and RHEL-based Linux distributions.

### Installation

Install the [Datasaker role][1] from Ansible Galaxy on your Ansible server:

```shell
ansible-galaxy install datasaker.datasaker
```

To deploy the Datasaker Agent on hosts, add the Datasaker role and your API key to your playbook:

```text
- hosts: servers
  roles:
    - { role: datasaker.datasaker, become: yes }
  vars:
    datasaker_api_key: "<YOUR_DD_API_KEY>"
```

#### Role variables

| Variable                                   | Description                                                                                                                                                                                                                                                                                                                                                        |
|--------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `datasaker_api_key`                          | Your Datasaker API key.                                                                                                                                                                                                                                                                                                                                              |
| `datasaker_site`                             | The site of the Datasaker intake to send Agent data to. Defaults to `datasakerhq.com`, set to `datasakerhq.eu` to send data to the EU site. This option is only available with Agent version >= 6.6.0.                                                                                                                                                                   |

| `datasaker_agent_major_version`              | The major version of the Agent to install. The possible values are 5, 6, or 7 (default). If `datasaker_agent_version` is set, it takes precedence otherwise the latest version of the specified major is installed. Setting `datasaker_agent_major_version` is not needed if `datasaker_agent_version` is used.                                                          |

### Integrations

To configure a Datasaker integration (check), add an entry to the `datasaker_checks` section. The first level key is the name of the check, and the value is the YAML payload to write the configuration file. Examples are provided below.

#### Process check

To define two instances for the `process` check use the configuration below. This creates the corresponding configuration files:

* `/etc/datasaker-agent/conf.yaml`

```yml
    datasaker_checks:
      process:
        init_config:
        instances:
          - name: ssh
            search_string: ['ssh', 'sshd']
          - name: syslog
            search_string: ['rsyslog']
            cpu_check_interval: 0.2
            exact_match: true
            ignore_denied_access: true
```

#### Custom check

To configure a custom check use the configuration below. This creates the corresponding configuration files:

* `/etc/datasaker-agent/conf.yaml`

```yml
    datasaker_checks:
      my_custom_check:
        init_config:
        instances:
          - some_data: true
```

### Tracing

To enable trace collection with Agent v6 or v7 use the following configuration:

```yaml
datasaker_config:
  apm_config:
    enabled: true
```

To enable trace collection with Agent v5 use the following configuration:

```yaml
datasaker_config:
  apm_enabled: "true" # has to be a string
```

### Live processes

To enable [live process][6] collection with Agent v6 or v7 use the following configuration:

```yml
datasaker_config:
  process_config:
    enabled: "true" # type: string
```

The possible values for `enabled` are: `"true"`, `"false"` (only container collection), or `"disabled"` (disable live processes entirely).

### Uninstallation

However for more control over the uninstall parameters, the following code can be used.
In this example, the '/norestart' flag is added and a custom location for the uninstallation logs is specified:

```yml
- name: Check If Datasaker Agent is installed
  win_stat:
  path: 'c:\Program Files\Datasaker\Datasaker Agent\bin\agent.exe'
  register: stat_file
- name: Uninstall the Datasaker Agent
  win_shell: start-process msiexec -Wait -ArgumentList ('/log', 'C:\\uninst.log', '/norestart', '/q', '/x', (Get-WmiObject -Class Win32_Product -Filter "Name='Datasaker Agent'" -ComputerName .).IdentifyingNumber)
  when: stat_file.stat.exists == True
```

## Troubleshooting

### Debian stretch

**Note:** this information applies to versions of the role prior to 4.9.0. Since 4.9.0, the `apt_key` module is no longer used by the role.

On Debian Stretch, the `apt_key` module used by the role requires an additional system dependency to work correctly. The dependency (`dirmngr`) is not provided by the module. Add the following configuration to your playbooks to make use of the present role:

```yml
---
- hosts: all
  pre_tasks:
    - name: Debian Stretch requires the dirmngr package to use apt_key
      become: yes
      apt:
        name: dirmngr
        state: present
  roles:
    - { role: datasaker.datasaker, become: yes }
  vars:
    datasaker_api_key: "<YOUR_DD_API_KEY>"
```
