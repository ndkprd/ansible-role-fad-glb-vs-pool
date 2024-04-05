# ansible-role-fortiadc-glb-vsp

## Description

Ansible role to create/update Fortinet's FortiADC GLB Virtual Server Pool.

Since VSP need GLB Servers, it needs my other role, [ndkprd.fortiadc-glb-servers](https://github.com/ndkprd/ansible-role-fortiadc-glb-servers), which in turn also need another role I made: [ndkprd.fortiadc-glb-data-center](https://github.com/ndkprd/ansible-role-fortiadc-glb-data-center).

## Usage

### Install Role

#### From Galaxy

```
ansible-galaxy install ndkprd.fortiadc-glb-vsp
```

#### From Github

##### Create Requirements File

```
---
# ./requirements.yaml

- name: ndkprd.fortiadc-glb-servers
  scm: git
  src: https://github.com/ndkprd/ansible-role-fortiadc-glb-vsp
  version: main
```

##### Install

```
ansible-galaxy install -r requirements.yaml
```

### Hosts Example

```
fad1.ndkprd.com fad_apitoken=mysupersecrettoken
fad2.ndkprd.com fad_apitoken=mysupersecrettoken
```

### Vars Example

```
---
fad_vdom: root

fad_glb_data_centers:
  - name: hq.ndkprd.com  # Data Center name/mkey
    location: ID  # 2 letters country ID
  - name: dc1.ndkprd.com    
    location: ID     
  - name: dc2.ndkprd.com
    location: ID

fad_glb_servers:
  - name: "dmz.hq.ndkprd.com" # GLB Server name/mkey
    data_center: "hq.ndkprd.com" # Data Center name, must exists first
    health_check_ctrl: enable
    health_check_list: "LB_HLTHCK_ICMP " # the whitespace is a must
    health_check_relationship: AND      
    server_type: Generic-Host # "Generic-Host" or "FortiADC-SLB"
    auth_type: none # Only for FortiADC-SLB
    address_type: ipv4 # FAD address type
    auto_sync: disable # Only for FortiADC-SLB
    fad_ipv4: "0.0.0.0" # Only for FortiADC-SLB
    fad_ipv6: "::" # Only for FortiADC-SLB
    fad_pass: "" # Only for FortiADC-SLB
    fad_port: "5858" # Only for FortiADC-SLB
    server_members: []

  - name: "dmz.dc1.ndkprd.com"
    data_center: "dc1.ndkprd.com"
    health_check_ctrl: enable
    health_check_list: "LB_HLTHCK_ICMP "
    health_check_relationship: AND
    server_type: Generic-Host
    auth_type: none
    address_type: ipv4 # FAD address type
    auto_sync: disable
    fad_ipv4: "0.0.0.0"
    fad_ipv6: "::"
    fad_pass: ""
    fad_port: "5858"
    server_members:
      - name: public-waf-1.dmz.dc1.ndkprd.com
        ipv4: 36.92.152.196
        ipv6: "::"
        address_type: ipv4
        gateway: ""
        health_check_ctrl: disable # default value
        health_check_inherit: enable
        health_check_list: ""
        health_check_relationship: OR # default value
      - name: public-waf-2.dmz.dc1.ndkprd.com
        ipv4: 114.5.127.186
        ipv6: "::"
        address_type: ipv4
        gateway: ""
        health_check_ctrl: disable # default value
        health_check_inherit: enable
        health_check_list: ""
        health_check_relationship: OR # default value

  - name: "dmz.dc2.ndkprd.com"
    data_center: "dc2.ndkprd.com"
    health_check_ctrl: enable
    health_check_list: "LB_HLTHCK_ICMP "
    health_check_relationship: AND
    server_type: Generic-Host
    auth_type: none
    address_type: ipv4 # FAD address type
    auto_sync: disable
    fad_ipv4: "0.0.0.0"
    fad_ipv6: "::"
    fad_pass: ""
    fad_port: "5858"
    server_members:
      - name: public-waf-1.dmz.dc2.ndkprd.com
        ipv4: 114.5.119.176
        ipv6: "::"
        address_type: ipv4
        gateway: ""
        health_check_ctrl: disable # default value
        health_check_inherit: enable
        health_check_list: ""
        health_check_relationship: OR # default value

fad_glb_vs_pools:
  - name: public-waf.dc1.ndkprd.com # VS Pools mkey
    check_server_status: enable # healthcheck
    check_virtual_server_existent: enable
    load_balance_method: wrr
    vs_pool_members:
      - id: 1001 # high number of ID for mkey
        is_backup: disable # if enable, when healthcheck failed it will goes to this server
        server: dmz.dc1.ndkprd.com # GLB Servers
        server_member_name: public-waf-1.dc1.ndkprd.com # GLB Servers member
        weight: 100
      - id: 1002
        is_backup: disable
        server:  dmz.dc1.ndkprd.com
        server_member_name: public-waf-2.dc1.ndkprd.com

```

### Playbook Example

```
---
# ./playbook.yaml

- name: Setup global-load-balance VSP entry in FortiADC.
  hosts: fortiadc
  become: false
  gather_facts: no
  vars_files:
    - ./fad-glb-vsp-vars.yaml

  roles:
    - ndkprd.fortiadc-glb-vsp

```

### About Tags

I added quiet lots of debug task, mainly to check if the variable I set is correct. These tags basically just print out the var that the previous task set/register. You can skip them altogether by skipping tasks with `debug` tags.

For example, if you're using CLI, you can just go `ansible-playbook playbook.yaml --skip-tags debug`.

As mentioned above, this role recursively depends on my two other FortiADC GLB role: one for GLB Data Center, other for GLB Servers and its members. You can use/skip them with the following tags:

- ndkprd.fortiadc-glb-data-center -> `fad-glb-dc`
- ndkprd.fortiadc-glb-servers -> `fad-glb-servers` and `fad-glb-servers-members`

## Limitation

Developed and tested against FortiADC 7.0.

## License

MIT, use at your own risk.