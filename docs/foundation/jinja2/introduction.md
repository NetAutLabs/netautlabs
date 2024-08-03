---
tags:
  - templating
  - jinja2
  - introduction
---

# Basic templates

The following exercises provide a basic introduction to Jinja2 templating.


To verify your solution, you can use an online Jinja sandbox like [Jinja101](https://jinja101.infrastructureascode.ch/) or the Python CLI tool [Nettowel](https://pypi.org/project/nettowel/).

!!! tip

    You can install "nettowel" with [`pipx`](https://pipx.pypa.io/stable/):
    ```bash
    pipx install nettowel[full]
    ``` 


## Find the input data

In this chapter, the goal of the exercise is to identify the YAML input used to render the configuration snippet with the provided template.

### VLAN

```jinja title="template"
!
{%- for vlan in vlans %}
vlan {{ vlan['id'] }}
 name {{ vlan['name'] | replace(' ', '_') }}
{%- endfor %}
!
```

``` title="rendered configuration"
!
vlan 10
 name hr
vlan 11
 name finance
vlan 21
 name printer
vlan 30
 name server_external
!
```

What YAML data was used to render the template?

??? Solution

    ```yaml
    ---
    vlans:
    - id: 10
      name: hr
    - id: 11
      name: finance
    - id: 21
      name: printer
    - id: 30
      name: server external
    ...
    ```

### System

```jinja title="template"
system {
    name-server {
    {%- for name_server in dns %}
        {{ name_server }};
    {%- endfor %}
    }
    syslog {
        {%- for syslog_server in syslog %}
        host {{ syslog_server }} {
            any any;
            daemon info;
        }
        {%- endfor %}
    }
    ntp {
        {%- for ntp_server in ntp %}
        server {{ ntp_server }};
        {%- endfor %}
    }
}
```

``` title="rendered configuration"
system {
    name-server {
        1.1.1.1;
        8.8.8.8;
        8.8.4.4;
    }
    syslog {
        host 10.0.0.10 {
            any any;
            daemon info;
        }
    }
    ntp {
        server 0.ch.pool.ntp.org;
        server 1.ch.pool.ntp.org;
    }
}
```

What YAML data was used to render the template?

??? Solution

    ```yaml
    ---
    dns:
      - 1.1.1.1
      - 8.8.8.8
      - 8.8.4.4
    ntp:
      - 0.ch.pool.ntp.org
      - 1.ch.pool.ntp.org
    syslog:
      - 10.0.0.10
    ...
    ```

### OSPF

```jinja title="template"
!
router ospf {{ ospf.id }}
{%- for net in ospf.network %}
 network {{ net.net }} area {{ net.area }}
{%- endfor %}
!
{%- for interface in ospf.interface %}
interface {{ interface.name }}
 ip ospf {{ ospf.id }} area {{ interface.area }}
 {%- if interface.cost is defined %}
 ip ospf {{ ospf.id }} cost {{ interface.cost }}
 {%- endif %}
 {%- if interface.priority is defined %}
 ip ospf priority {{ interface.priority }}
 {%- endif %}
!
{%- endfor %}
```

``` title="rendered configuration"
!
router ospf 1
 network 192.168.0.0 0.0.255.255 area 0
 network 172.16.10.0 0.0.0.255 area 1
!
interface Lo0
 ip ospf 1 area 0
!
interface Ethernet 0
 ip ospf 1 area 0
 ip ospf 1 cost 10
!
interface Ethernet 3
 ip ospf 1 area 0
 ip ospf priority 9
!
```

What YAML data was used to render the template?

??? Solution

    ```yaml
    ---
    ospf:
      id: 1
      network:
        - net: 192.168.0.0 0.0.255.255
          area: 0
        - net: 172.16.10.0 0.0.0.255
          area: 1
      interface:
        - name: Lo0
          area: 0
        - name: Ethernet 0
          area: 0
          cost: 10
        - name: Ethernet 3
          area: 0
          priority: 9
    ...
    ```

## Write the template

In this chapter, the goal of the exercise is to write a Jinja2 template that produces the expected configuration snippet using the provided input data.

### VLAN

``` yaml title="input data"
---
vlans:
  -
    - 10
    - hr
  -
    - 11
    - finance
  -
    - 21
    - printer
  -
    - 30
    - server external
```

``` title="rendered configuration"
!
vlan 10
 name hr
vlan 11
 name finance
vlan 21
 name printer
vlan 30
 name server_external
!
```

What template was used to generate the configuration snippet?

??? Solution

    ```jinja
    !
    {%- for vlan in vlans %}
    vlan {{ vlan[0] }}
    name {{ vlan[1] | replace(' ', '_') }}
    {%- endfor %}
    !
    ```

### System

```yaml title="input data"
---
server:
  dns:
    - 1.1.1.1
    - 8.8.8.8
    - 8.8.4.4
  ntp:
    - 0.ch.pool.ntp.org
    - 1.ch.pool.ntp.org
  syslog:
    - 10.0.0.10
```

``` title="rendered configuration"
!
ip name-server 1.1.1.1 8.8.8.8 8.8.4.4
!
service timestamps log datetime msec
logging buffered 8192
logging host 10.0.0.10
!
ntp server 0.ch.pool.ntp.org
ntp server 1.ch.pool.ntp.org
!
```

What template was used to generate the configuration snippet?

??? Solution

    ```jinja
    !
    ip name-server{% for name_server in server['dns'] %} {{ name_server }}{% endfor %}
    !
    service timestamps log datetime msec
    logging buffered 8192
    {%- for syslog_server in server['syslog'] %}
    logging host {{ syslog_server }}
    {%- endfor %}
    !
    {%- for ntp_server in server['ntp'] %}
    ntp server {{ ntp_server }}
    {%- endfor %}
    !
    ```

### Interface

```yaml title="input data"
---
interfaces:
  - name: ge-0/0/2
    description: uplink
    ip: 192.168.1.1
    mask: 30
  - name: Lo0
    ip: 10.10.10.10
    mask: 32
```

``` title="rendered configuration"
interfaces {
    ge-0/0/2 {
        description uplink;
        unit 0 {
            family inet {
                address 192.168.1.1/30;
            }
        }
    }
}
interfaces {
    Lo0 {
        unit 0 {
            family inet {
                address 10.10.10.10/32;
            }
        }
    }
}
```

What template was used to generate the configuration snippet?

??? Solution

    ```jinja
    {%- for interface in interfaces %}
    interfaces {
        {{ interface['name'] }} {
            {%- if interface['description'] is defined %}
            description {{ interface['description'] }};
            {%- endif %}
            unit 0 {
                family inet {
                    address {{ interface['ip'] }}/{{ interface['mask'] }};
                }
            }
        }
    }
    {%- endfor %}
    ```

## Render the templates

In the following exercises, you will take on the role of Jinja2 and render the templates using the provided input data.

### VLAN

```jinja title="template"
!
{%- for id, name in vlans.items() %}
vlan {{ id }}
 name {{ name | replace(' ', '_') }}
{%- endfor %}
!
```

``` yaml title="input data"
---
vlans:
  10: hr
  11: finance
  12: server external
```

What does the generated configuration snippet look like?

??? Solution

    ```
    !
    vlan 10
    name hr
    vlan 11
    name finance
    vlan 12
    name server_external
    !
    ```

### System

```jinja title="template"
!
ip name-server{% for name_server in servers[0] %} {{ name_server }}{% endfor %}
!
service timestamps log datetime msec
logging buffered 8192
{%- for syslog_server in servers[2] %}
logging host {{ syslog_server }}
{%- endfor %}
!
{%- for ntp_server in servers[1] %}
ntp server {{ ntp_server }}
{%- endfor %}
!
```

``` yaml title="input data"
---
servers:
  - [1.1.1.1, 8.8.8.8, 8.8.4.4]
  - [0.ch.pool.ntp.org, 1.ch.pool.ntp.org]
  - [10.0.0.10]
```

What does the generated configuration snippet look like?

??? Solution

    ```
    !
    ip name-server 1.1.1.1 8.8.8.8 8.8.4.4
    !
    service timestamps log datetime msec
    logging buffered 8192
    logging host 10.0.0.10
    !
    ntp server 0.ch.pool.ntp.org
    ntp server 1.ch.pool.ntp.org
    !
    ```

### VRF

```jinja title="template"
{%- for vrf in vrfs %}
routing-instances {
    {{ vrf.name }} {
        instance-type vrf;
        route-distinguisher {{ asn }}:{{ vrf.id }}
        vrf-target target:{{ asn }}:123456;
        protocols {
            bgp {
                group {{ vrf.name }} {
                    log-updown;
                    {%- for neighbor in bgp[vrf.name] %}
                    neighbor {{ neighbor.ip }} {
                        description {{ neighbor.name }};
                        peer-as {{ neighbor.asn }};
                    }
                    {%- endfor %}
                }
            }
        }
    }
}
{%- endfor %}
```

``` yaml title="input data"
---
asn: 65000

vrfs:
  - name: CustA
    id: 123
  - name: CustB
    id: 124

bgp:
  CustA:
    - ip: 10.0.0.1
      name: CE01
      asn: 65010
    - ip: 10.0.0.5
      name: CE02
      asn: 65010
  CustB:
    - ip: 192.168.0.1
      name: CE01
      asn: 65064
    - ip: 192.168.0.5
      name: CE02
      asn: 65064
```

What does the generated configuration snippet look like?

??? Solution

    ```
    routing-instances {
        CustA {
            instance-type vrf;
            route-distinguisher 65000:123
            vrf-target target:65000:123456;
            protocols {
                bgp {
                    group CustA {
                        log-updown;
                        neighbor 10.0.0.1 {
                            description CE01;
                            peer-as 65010;
                        }
                        neighbor 10.0.0.5 {
                            description CE02;
                            peer-as 65010;
                        }
                    }
                }
            }
        }
    }
    routing-instances {
        CustB {
            instance-type vrf;
            route-distinguisher 65000:124
            vrf-target target:65000:123456;
            protocols {
                bgp {
                    group CustB {
                        log-updown;
                        neighbor 192.168.0.1 {
                            description CE01;
                            peer-as 65064;
                        }
                        neighbor 192.168.0.5 {
                            description CE02;
                            peer-as 65064;
                        }
                    }
                }
            }
        }
    }
    ```
