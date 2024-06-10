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
domain_name: "aoshouston.org"
search_domain: "aoshouston.org"
dns_servers:
  - 8.8.8.8
  - 1.1.1.1
ansible_user: username
ansible_ssh_pass: password
ansible_become: true
ansible_network_os: arubanetworks.aoscx.aoscx
ansible_connection: network_cli
management_vlan: 99
gateway: 1.2.3.4
mstp_name: mstpname
mstp_revision: mstprevision
port_access:
  lldp_group: Aruba-IAP-Group
  role: Aruba-IAP-role
  profile: Aruba-IAP-Prof
  description: Aruba IAP
  native_vlan: 105
  allowed_vlan: 16,17,19,21,24
vlans:
  - { id: '2', name: 'Voice', type: voice }
  - { id: '16', name: 'Aruba-DMZ1', type: "" }
  - { id: '17', name: 'Step1', type: "" }
  - { id: '19', name: 'Aruba-DMZ2', type: "" }
  - { id: '20', name: 'IPAD', type: "" }
  - { id: '21', name: 'Step3', type: "" }
  - { id: '23', name: 'IPADMINI', type: "" }
  - { id: '24', name: 'CBST', type: "" }
  - { id: '99', name: 'SW-MGMT', type: "" }
  - { id: '100', name: 'MSBasement', type: "" }
  - { id: '105', name: 'AP-MGMT', type: "" }
  - { id: '110', name: 'TeachStaff1', type: "" }
  - { id: '120', name: 'TeachStaff2', type: "" }
  - { id: '130', name: 'MS3rdFloor', type: "" }
  - { id: '150', name: 'Printers', type: "" }
  - { id: '175', name: 'OOB-PUBSW', type: "" }
  - { id: '200', name: 'TeachStaff3', type: "" }
  - { id: '800', name: 'Trunk_Native', type: "" }
  - { id: '900', name: 'BLACK_HOLE', type: "" }
```

#### Example `host_vars/switch1.yml`

```yaml
hostname: hostname
management_ip_cidr: 10.99.99.2/24
mstp_priority: 0
vsf:
  - { member: 1, type: 'r8q69a', link_1: 1/1/49, link_2: 1/1/50}
  - { member: 2, type: 'r8q70a', link_1: 1/1/49, link_2: 2/1/50}
  - { member: 3, type: 'r8q70a', link_1: 1/1/49, link_2: 3/1/50}
  - { member: 4, type: 'r8q70a', link_1: 1/1/49, link_2: 4/1/50}
  - { member: 5, type: 'r8q70a', link_1: 1/1/49, link_2: 5/1/50}
vsf_secondary_member: 5
voice_interfaces:
  - { port: '1/1/25-1/1/48,2/1/1,2/1/3,2/1/6,2/1/8-2/1/13,2/1/15,2/1/17-2/1/23,2/1/25-2/1/35,2/1/37,2/1/41,2/1/43-2/1/47,3/1/1-3/1/4,3/1/7,3/1/9-3/1/22,3/1/24-3/1/32,3/1/35-3/1/46,4/1/1-4/1/48,5/1/17-5/1/48', allowed_vlans: '2', native_vlan: '200', description: 'Voice Port' }
trunk_interfaces:
  - { port: '1/1/51-1/1/52,2/1/51-2/1/52,3/1/51-3/1/52,4/1/51-4/1/52,5/1/51-5/1/52', allowed_vlans: '2,16,17,19,20,21,23,24,99,100,105,110,120,130,150,175,200', native_vlan: '800', description: 'Trunk Port' }
access_interfaces:
  - { port: '1/1/1-1/1/24', vlan: '1', description: '' }
  - { port: '2/1/2,2/1/4,2/1/14,2/1/16,2/1/24,2/1/36,2/1/38,2/1/39,2/1/42,2/1/48,3/1/5-3/1/6,3/1/7,3/1/9-3/1/22,3/1/24-3/1/32,3/1/35-3/1/46', vlan: '20', description: '' }
  - { port: '2/1/5,2/1/7,3/1/47-3/1/48', vlan: '1', description: '' }
  - { port: '5/1/1-5/1/16', vlan: '200', description: '' }
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
dhcpv4-snooping
{% if vlans %}
{% for vlan in vlans -%}
vlan {{ vlan.id }}
{%- if vlan.name %}    
    name {{ vlan.name }}
{% if vlan.type %}
    {{ vlan.type }}
{% endif %}
{% endif %}
{% endfor %}
{%- endif %}
{% if port_access %}
port-access lldp-group {{ port_access.lldp_group }}
     seq 10 match sys-desc IAP
port-access role {{ port_access.role }}
    description {{ port_access.description }}
    poe-priority critical
    vlan trunk native {{ port_access.native_vlan }}
    vlan trunk allowed {{ port_access.allowed_vlan }},{{ port_access.native_vlan }}
port-access device-profile Aruba-IAP-Prof
    enable
    associate role {{ port_access.role }}
    associate lldp-group {{ port_access.lldp_group }}
{% endif %}
spanning-tree mode mstp
spanning-tree config-name {{ mstp_name }}
spanning-tree config-revision {{ mstp_revision }}
spanning-tree priority {{ mstp_priority }}

{% if voice_interfaces %}
{% for interface in voice_interfaces %}
interface {{ interface.port }}
    vlan trunk allowed {{ interface.allowed_vlans }}{% if interface.native_vlan %},{{ interface.native_vlan }}{% endif %}

{% if interface.native_vlan %}
    vlan trunk native {{ interface.native_vlan }}
{% endif %}
{% if interface.description %}
    description {{ interface.description }}
{% endif %}
    spanning-tree bpdu-guard
    spanning-tree port-type admin-edge
{% endfor %}
{% endif %}
{% if trunk_interfaces %}
{% for interface in trunk_interfaces %}
interface {{ interface.port }}
    vlan trunk allowed {{ interface.allowed_vlans }}{% if interface.native_vlan %},{{ interface.native_vlan }}{% endif %}

{% if interface.native_vlan %}
    vlan trunk native {{ interface.native_vlan }}
{% endif %}
{% if interface.description %}
    description {{ interface.description }}
{% endif %}
    dhcpv4-snooping trust
{% endfor %}
{% endif %}
{% if access_interfaces %}
{% for interface in access_interfaces %}
interface {{ interface.port }}
    vlan access {{ interface.vlan }}
    spanning-tree bpdu-guard
    spanning-tree port-type admin-edge
{% if interface.description %}
    description {{ interface.description }}
{% endif %}
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
{% if gateway %}
ip route 0.0.0.0/0 {{ gateway }}
{% endif %}
{% if vsf %}
{% for vsf in vsf %}
vsf member {{ vsf.member }}
    type {{ vsf.type }}
    link 1 {{ vsf.link_1 }}
    link 2 {{ vsf.link_2 }}
{% endfor %}
{% endif %}
{% if vsf_secondary_member %}
vsf secondary-member {{ vsf_secondary_member }}
{% endif %}
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