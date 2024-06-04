## Ansible Role: aoscx_config_template

### Overview

This Ansible role is designed to allow configuration to be generated from defined variables and then pushed to switches.

### Requirements

- Ansible 2.9+ (preferably the latest version)
- AOS-CX switches with SSH access enabled
- Python modules: `paramiko` and `netmiko`

### Role Variables

#### Required Variables

These variables must be defined either in `host_vars`, `group_vars`, or `inventory`:

- `ansible_user`: Username to access the switch.
- `ansible_password`: Password for the user.
- `ansible_become`: Boolean indicating if privilege escalation is needed.
- `ansible_become_method`: Method to use for privilege escalation (e.g., `enable`).
- `ansible_become_password`: Password for privilege escalation.
- `ansible_network_os`: arubanetworks.aoscx.aoscx
- `ansible_connection`: network_cli #arubanetworks.aoscx.aoscx

#### Config Variables
- `vlans`: List of VLANs to be configured.
- `id`: VLAN ID.
- `name`: VLAN name (optional)
- `trunk_interfaces`: List of trunk interfaces to be configured.
- `port`: Interface port.
- `allowed_vlans`: VLANs allowed on this trunk.
- `native_vlan`: Native VLAN for this trunk.
- `access_interfaces`: List of access interfaces to be configured.
- `port`: Interface port.
- `vlan`: VLAN assigned to this access interface.
- `management_vlan`: Management VLAN ID.
- `management_ip_cidr`: Management IP address with CIDR notation.
- `dns_servers`: List of DNS server IP addresses.
- `search_domain`: DNS search domain.
- `hostname`: Hostname of the switch.
- `domain_name`: Domain name of the switch.

### Directory Structure

```
ansible_project/
├── aoscx_template_config/
│   ├── LICENSE
│   ├── README.md
│   ├── files/
│   ├── meta/
│   │   └── main.yml
│   ├── tasks/
│   │   ├── apply_config.yml
│   │   ├── generate_config.yml
│   │   └── main.yml
│   └── templates/
│       └── aoscx_config.j2
├── group_vars/
├── host_vars/
├── inventory.yml
└── playbook.yml
```

### Using `host_vars` and `group_vars` folders

Both `group_vars` and `host_vars` folders will be automatically scanned by Ansible in the directory the inventory file was loaded from

`group_vars` - File names must match groups defined within inventory file.
`host_vars` - File names must match hosts defined within inventory file.

### Note: The variables in the following examples can be either group or host variables.  Use them as necessary for your project

#### Example `group_vars/all.yml`

```yaml
ansible_user: admin
ansible_password: your_password
ansible_become: yes
ansible_become_method: enable
ansible_become_password: your_enable_password
dns_servers:
  - "8.8.8.8"
  - "8.8.4.4"
domain_name: "exmaple.com"
search_domain: "example.com"
vlans:
  - id: '10'
    name: 'test1'
  - id: '11'
    name: 'test2'
management_vlan: 99
```

#### Example `host_vars/switch1.yml`

```yaml
hostname: switch1
management_ip_cidr: '192.168.1.2/24'
trunk_interfaces:
  - port: '1/1/6'
    allowed_vlans: '10,11,12'
    native_vlan: '99'
access_interfaces:
  - port: '1/1/3'
    vlan: '10'
  - port: '1/1/4'
    vlan: '11'
```

#### Example `host_vars/switch2.yml`

```yaml
hostname: switch2
management_ip_cidr: '192.168.1.3/24'
```

### Jinja2 Template

The Jinja2 template `aoscx_config.j2` is used to generate the configuration file. It supports conditional inclusion of configuration elements based on the provided variables.  Please use this as an example to build upon this by adding additional conditional config.

```jinja
{% if hostname %}
hostname {{ hostname }}
{% endif %}
{% if domain_name %}
domain-name {{ domain_name }}
{% endif %}
{% if vlans %}
{% for vlan in vlans -%}
vlan {{ vlan.id }}
{%- if vlan.name %}    
    name {{ vlan.name }}
{% endif %}
{% endfor %}
{%- endif %}
{% if trunk_interfaces %}
{% for interface in trunk_interfaces -%}
interface {{ interface.port }}
    vlan trunk allowed {{ interface.allowed_vlans }}{% if interface.native_vlan %},{{ interface.native_vlan }}{% endif %}
{% if interface.native_vlan -%}
    vlan trunk native {{ interface.native_vlan }}
{% endif %}
{% endfor %}
{% endif %}
{% if access_interfaces %}
{% for interface in access_interfaces %}
interface {{ interface.port }}
    vlan access {{ interface.vlan }}
{% endfor %}
{% endif %}
{% if management_vlan and management_ip_cidr %}
interface vlan {{ management_vlan }}
    ip address {{ management_ip_cidr }}
{% endif %}
{% if search_domain %}
ip dns domain-name {{ search_domain }}
{% endif %}
{% if dns_servers -%}
{% for server in dns_servers -%}
ip dns server-address {{ server }}
{% endfor -%}
{% endif -%}
```

### Playbook Example

Here’s an example playbook that uses this role:

```yaml
- name: Apply AOS-CX configuration
  hosts: aoscx-switches
  gather_facts: no
  roles:
    - aoscx_template_config
```

### Running the Playbook

To run the playbook, use the following command:

```bash
ansible-playbook -i inventory playbook.yml
```