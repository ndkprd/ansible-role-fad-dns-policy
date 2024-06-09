# ansible-role-fortiadc-dns-policy

## Description

Ansible role to create/update Fortinet's FortiADC Global DNS Policy entries and General poliicy via their HTTP REST API. 

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
    # global-dns-server address-group entries
    fad_dns_address_groups:
      - name: local
        address_group_members:
          - id: 1
            addr_type: ipv4
            ip6_network: "::/0"
            ip4_network: "10.0.0.0/8"
            action: "include"
          - id: 2
            addr_type: ipv4
            ip6_network: "::/0"
            ip4_network: "172.16.0.0/12"
            action: "include"
          - id: 3
            addr_type: ipv4
            ip6_network: "::/0"
            ip4_network: "192.168.0.0/16"
            action: "include"
    # global-dns-server policy
    fad_dns_policies:
      - name: "LOCAL_DNS_POLICY" # Global DNS Policy mkey
        source_address: "local" # valid Address Group entry mkey used as source
        destination_address: "local" # valid Address Group entry used as destination
        dns64_list: ""
        dnssec_validate_status: "disable" # "enable" or "disable"
        forward: "first" # "first" or "only"
        forwarders: "" # valid Remote DNS Servers entry mkey
        recursion_status: "disable" # "enable" or "disable"
        rrlimit: "" # valid Response Rate Limit 
      - name: "PUBLIC_DNS_POLICY" # Global DNS Policy mkey
        source_address: "any" # valid Address Group entry mkey used as source
        destination_address: "any" # valid Address Group entry used as destination
        dns64_list: ""
        dnssec_validate_status: "disable" # "enable" or "disable"
        forward: "first" # "first" or "only"
        forwarders: "" # valid Remote DNS Servers entry mkey
        recursion_status: "disable" # "enable" or "disable"
        rrlimit: "" # valid Response Rate Limit 
    fad_dns_general_settings:
      global_dns_mode: "enable"
      certificate: "Factory"
      dnssec_validate_status: "disable"
      doh_interface_list: ""
      doh_port: "80"
      doh_status: "disable"
      dohs_interface_list: ""
      dohs_port: "443"
      dohs_status: "disable"
      dot_interface_list: ""
      dot_port: "853"
      dot_status: "disable"
      forward: "first"
      forwarders: ""
      interface_list: ""
      ipv4_mode: "enable"
      ipv6_mode: "disable"
      listen_to_all_interface: "enable"
      recursion_status: "disable"
      response_ratelimit: ""
      traffic_log: "enable"
      use_system_dns_server: "enable"

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