# ansible-role-fortiadc-glb-vs-pool

## Description

Ansible role to create/update Fortinet's FortiADC GLB Virtual Server Pools and its member.

## Usage

### Install Role

```
ansible-galaxy install ndkprd.fad_glb_vs_pools
```

### Hosts Example

```
fad1.ndkprd.com fad_apitoken=mysupersecrettoken fad_vdom=root
```

### Playbook Example

```
---
# ./playbook.yaml

- name: Update/create FortiADC resources.
  hosts: all
  become: false
  gather_facts: false
  connection: local
  vars:
     # for testing-purpose only, to delete the created resources after. change to 'false' or override from --extra-vars if you want it to stay.
    do_cleanup: true
    # global-load-balance data-center entries
    fad_glb_data_centers:
      - name: dc1.ndkprd.com
        location: ID
    # global-load-balance servers entries
    fad_glb_servers:
      - name: "dmz.dc1.ndkprd.com"
        data_center: "dc1.ndkprd.com"
        health_check_ctrl: enable
        health_check_list: "LB_HLTHCK_ICMP "
        health_check_relationship: OR
        server_type: Generic-Host
        auth_type: none
        address_type: ipv4 # FAD address type
        auto_sync: disable
        fad_ipv4: "0.0.0.0"
        fad_ipv6: "::"
        fad_pass: ""
        fad_port: "5858"
        server_members:
          - name: public-waf-1.dc1.ndkprd.com
            ipv4: 10.10.1.1
            ipv6: "::"
            address_type: ipv4
            gateway: ""
            health_check_ctrl: disable
            health_check_inherit: enable
            health_check_list: ""
            health_check_relationship: "OR"
    # global-load-balance virtual-server-pool entries
    fad_glb_vs_pools:
      - name: public-waf.dc1.ndkprd.com # VS Pools mkey
        check_server_status: enable # healthcheck
        check_virtual_server_existent: disable
        load_balance_method: wrr
        vs_pool_members:
          - id: 1001 # high number of ID for mkey
            is_backup: disable # if enable, when healthcheck failed it will goes to this server
            server: dmz.dc1.ndkprd.com # GLB Servers
            server_member_name: public-waf-1.dc1.ndkprd.com # GLB Servers member
            weight: 10

  roles:
    - ndkprd.fad_glb_vs_pool

```

### About Tags

Each task is tagged with their task file name, `fad_glb_vs_pool` and `fad_glb_vs_pool_member`. 

There's also a some kind of pre-task and post-task that run before and after create/update tasks to compare the value of before and after create/update tasks, tagged with the above tag and an additional `debug` tag.

## Limitation

Developed against and only tested for FortiADC 7.0.

## License

MIT, use at your own risk.
