---
tags:
  - data-serialization
  - yaml
  - introduction
---

# YAML Introduction

YAML (YAML Ain't Markup Language) is a human-readable data serialization standard that is commonly used for configuration files and data exchange between languages with different data structures. It will be extremely useful in your network automation projects. If you are not familiar with YAML, there are many excellent tutorials and resources available.

- [https://yaml.org/](https://yaml.org/)



The following exercises provide a basic introduction to YAML. To verify your solution, you can use online converters or CLI tools like [`yq`](https://github.com/mikefarah/yq) or [Nettowel](https://pypi.org/project/nettowel/). When working with YAML keep in mind the difference between version 1.1 and 1.2! `yq` and `nettowel` use version 1.2 as default. 


!!! tip

    You can install "nettowel" with [`pipx`](https://pipx.pypa.io/stable/):
    ```bash
    pipx install nettowel[full]
    ``` 

## JSON to YAML

- Convert the following JSON to YAML:

```json title="json"
{
    "host": "Switch01",
    "ip": "10.11.12.13",
    "dns": "8.8.8.8",
    "domain_name": "network.automation.lab",
    "fqdn": "Switch01.network.automation.lab"
}
```

??? Solution

    ```yaml
    ---
    host: Switch01
    ip: 10.11.12.13
    dns: 8.8.8.8
    domain_name: network.automation.lab
    fqdn: Switch01.network.automation.lab
    ...
    ```

- Convert the following JSON to YAML:

```json title="json"
[
    "interface Gig1/0/1",
    "interface Gig1/0/2",
    "interface gig1/0/3"
]
```

??? Solution

    ```yaml
    ---
    - interface Gig1/0/1
    - interface Gig1/0/2
    - interface gig1/0/3
    ...
    ```

- Convert the following JSON to YAML:

```json title="json"
{
    "OSPF": {
        "area": 10,
        "hello": 5,
        "interface-type": "p2p"
    },
    "bgp": {
        "asn": 65123,
        "rr": true
    }
}
```

??? Solution

    ```yaml
    ---
    OSPF: 
      area: 10
      hello: 5
      interface-type: p2p
    bgp:
      asn: 65123
      rr: True
    ...
    ```

- Convert the following JSON to YAML:

```json title="json"
[
    [
        11,
        "hr"
    ],
    [
        21,
        "finance"
    ],
    [
        31,
        "printer"
    ]
]
```

??? Solution

    ```yaml
    ---
    -
      - 11
      - hr
    -
      - 21
      - finance
    -
      - 31
      - printer
    ...
    ```

- Convert the following JSON to YAML:

```json title="json"
{
    "ospf": {
        "area": 0,
        "interfaces": [
        {
            "name": "Gig1/0/1",
            "passive": false,
            "type": "p2p"
        },
        {
            "name": "Gig1/0/1",
            "passive": true,
            "type": "broadcast"
        }
        ],
        "redistribute": [
        "bgp",
        "isis"
        ],
        "id": "10.10.10.10"
    }
}
```

??? Solution

    ```yaml
    ---
    ospf:
      area: 0
      interfaces:
        - name: Gig1/0/1
          passive: False
          type: p2p
        - name: Gig1/0/1
          passive: True
          type: broadcast
      redistribute:
        - bgp
        - isis
      id: '10.10.10.10'
    ...
    ```

## Craft YAML

- Create a sequence of mappings for vlan definitions with `id` and `name` as keys.

??? Solution

    ```yaml
    ---
    - name: hr
      id: 11
    - name: finance
      id: 22
    - name: printer
      id: 33
    - name: server
      id: 100
    ...
    ```

- Create a mapping for interfaces with tagged and untagged vlan sequences.

??? Solution

    ```yaml
    ---
    Ethernet 1/0/1:
      tagged: [10, 11, 12, 20, 21]
      untagged: [1]
    Ethernet 1/0/2:
      tagged: [13, 14, 15, 20]
      untagged: [1]
    Ethernet 1/0/3:
      tagged: []
      untagged: [20]
    ...
    ```

- Write a YAML file with `hostname`, `domain-name`, a list of `dns` servers, a list of `ntp` servers and a list of `syslog` servers.

??? Solution

    ```yaml
    ---
    hostname: sw01
    domain-name: network.automation.lab
    dns:
      - 1.1.1.1 # CloudFlare
      - 8.8.8.8 # Google DNS
    ntp:
      - 0.ch.pool.ntp.org
      - 1.ch.pool.ntp.org
    syslog:
      - 10.0.0.11
    ...
    ```

## YAML to JSON

- Convert the following YAML to JSON:

```yaml title="yaml"
%YAML 1.1
---
sexagesimal: 123:10:10
port: 80
alt_port: !!str 8080
octal: 02472256
hexadecimal: 0x_0A_74_AE
binary: 0b1010_0111_0100_1010_1110
...
```

??? Solution

    ```json
    {
      "sexagesimal": 443410,
      "port": 80,
      "alt_port": "8080",
      "octal": 685230,
      "hexadecimal": 685230,
      "binary": 685230,
    }
    ```


- Convert the following YAML to JSON:

```yaml title="yaml"
%YAML 1.2
---
sexagesimal: 123:10:10
port: 80
alt_port: !!str 8080
octal: 02472256
hexadecimal: 0x_0A_74_AE
binary: 0b1010_0111_0100_1010_1110
...
```

??? Solution

    ```json
    {
      "sexagesimal": "123:10:10",
      "port": 80,
      "alt_port": "8080",
      "octal": 2472256,
      "hexadecimal": 685230,
      "binary": 685230
    }
    ```



- Convert the following YAML to JSON:

```yaml title="yaml"
%YAML 1.1
---
'on': on
'off': off
'yes': yes
'no': no
'~': ~ 
'none': none
...
```

??? Solution

    ```json
    {
      "no": false,
      "none": "none",
      "off": false,
      "on": true,
      "yes": true,
      "~": null
    }
    ```

- Convert the following YAML to JSON:

```yaml title="yaml"
%YAML 1.2
---
'on': on
'off': off
'yes': yes
'no': no
'~': ~ 
'none': none
...
```

??? Solution

    ```json
    {
      "on": "on",
      "off": "off",
      "yes": "yes",
      "no": "no",
      "~": null,
      "none": "none"
    }
    ```


??? tip

    Always use `True|true` or `False|false` for booleans.

- Convert the following YAML to JSON:

```yaml title="yaml"
---
name: &a Network Automation Labs
alias: *a
mgmt_vlan: &mgmt_vlan
  name: mgmt
  id: 4
access_vlan: &access_vlan
  name: access
  id: 123
vlans:
  - *mgmt_vlan
  - *access_vlan
...
```

??? Solution

    ```json
    {
      "name": "Network Automation Labs",
      "alias": "Network Automation Labs",
      "mgmt_vlan": {
        "name": "mgmt",
        "id": 4
      },
      "access_vlan": {
        "name": "access",
        "id": 123
      },
      "vlans": [
        {
          "name": "mgmt",
          "id": 4
        },
        {
          "name": "access",
          "id": 123
        }
      ]
    }
    ```

- Convert the following YAML to JSON:

```yaml title="yaml"
---
CustA: &service
  name: CustA
  redistribute: True
  ipv6: True
  ipv4: True
CustB:
  <<: *service
  name: CustB
...
```

??? Solution

    ```json
    {
      "CustA": {
        "name": "CustA",
        "redistribute": true,
        "ipv6": true,
        "ipv4": true
      },
      "CustB": {
        "name": "CustB",
        "redistribute": true,
        "ipv6": true,
        "ipv4": true
      }
    }
    ```

