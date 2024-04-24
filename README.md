# ansible-role-fortiadc-dns-policy

## Description

Ansible role to create/update Fortinet's FortiADC Global DNS Policy via their HTTP REST API. 

## Prerequisite

The DNS Policy API need some other resources to be exists first: *Address Group*, *Remote DNS Servers*, *Response Rate Limit*, ~~and all the DNS Zones that you will put inside the `zone_list`~~. At this point in time I haven't created the roles to create/update those resources (and I don't think I will), so if you want to use customized resource for those, you need to make sure your customized resources entry you put in the variable is already exists.

## Usage

### Install Role

```
ansible-galaxy install ndkprd.fad_dns_policy
```

### Hosts Example

```
[fortiadc]
fad1.ndkprd.com fad_apitoken=mysupersecrettoken
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
    - ndkprd.fad_dns_policy

```

### About Tags

I added quiet lots of debug task, mainly to check if the variable I set is correct. These tags basically just print out the var that the previous task set/register. You can skip them altogether by skipping tasks with `debug` tags.

For example, if you're using CLI, you can just go `ansible-playbook playbook.yaml --skip-tags debug`.

## Limitation

Developed and tested against Hardware FortiADC 7.0 only.

About `zone_list`, since I haven't found a way to correlate which zones uses which DNS Policy, currently I basically just extract all existing Zones right before the PUT task, and then put them on the first policy (and since I basically just use `DEFAULT_DNS_POLICY`, it basically tell this policy to manage all the zones---which is fine for me, since I do split-horizon based on VDOM and not policy).

Make sure to run all other zone-making tasks/roles (GLB Hosts, DNS Zones) before running this roles if you want to add new zones to the policy automatically, or else it would be skipped. I didn't add those two roles as dependencies for the cases where you would want to just update the Policy data.

It's probably possible to separate the zones to separate policy, but that kinda makes this role can't be used on its own when not used with either GLB Host or DNS Zone roles, so this is the kind of thing I sacricified for the sake of modularity.

Though to be fair, you can still do it if you run this role more than one time. Like:

First round:

1. Create only zones that would be managed by policy1
2. add policy1 to the var
3. Run the policy role

Second Round:

1. Create new zones that would be managed by policy2
2. add policy2 to the var, above the policy1
3. Run the policy role

## License

MIT, use at your own risk.