# ansible-role-fortiadc-dns-policy

## Description

Ansible role to create/update Fortinet's FortiADC Global DNS Policy via their HTTP REST API. 

## Prerequisite

The DNS Policy API need some other resources to be exists first: *Address Group*, *Remote DNS Servers*, *Response Rate Limit*, ~~and all the DNS Zones that you will put inside the `zone_list`~~. At this point in time I haven't created the roles to create/update them, so if you want to use customized resource for those, you need to make sure your customized resources entry you put in variable is already exists.

## Usage

### Install Role

#### From Galaxy

```
ansible-galaxy install ndkprd.fortiadc-dns-policy
```

#### From Github

##### Create Requirements File

```
---
# ./requirements.yaml

- name: ndkprd.fortiadc-dns-policy
  scm: git
  src: https://github.com/ndkprd/ansible-role-fortiadc-dns-policy.git
  version: main # or 'devel' or release/tag name
```

##### Install

```
ansible-galaxy install -r requirements.yaml
```

### Hosts Example

```
[fortiadc]
fad1.ndkprd.com fad_apitoken=mysupersecrettoken
fad2.ndkprd.com fad_apitoken=mysupersecrettoken
```

### Playbook Example

```
---
# ./playbook.yaml

- name: Setup global-load-balance VSP entry in FortiADC.
  hosts: fortiadc
  become: false
  gather_facts: no
  vars:
    fad_vdom: "root"
    fad_dns_policy:
      - name: "DEFAULT_DNS_POLICY" # Global DNS Policy mkey
        source_address: "any" # valid Address Group entry mkey used as source
        destination_address: "any" # valid Address Group entry used as destination
        dns64_list: ""
        dnssec_validate_status: "disable" # "enable" or "disable"
        forward: "first" # "first" or "only"
        forwarders: "" # valid Remote DNS Servers entry mkey
        recursion_status: "disable" # "enable" or "disable"
        rrlimit: "" # valid Response Rate Limit 
        zone_list: "ndkprd.com devops.ndkprd.com infra.ndkprd.com " # list of zones, ended with space before quotation, must be a valid zone entry mkey

  roles:
    - ndkprd.fortiadc-glb-vsp

```

### About Tags

I added quiet lots of debug task, mainly to check if the variable I set is correct. These tags basically just print out the var that the previous task set/register. You can skip them altogether by skipping tasks with `debug` tags.

For example, if you're using CLI, you can just go `ansible-playbook playbook.yaml --skip-tags debug`.

## Limitation

Developed and tested against Hardware FortiADC 7.0 only.

Previously this task also manage `zone_list`, but it's kinda hard considering their behavior where the latest "list" is applied regardless of the existing condition, which is kinda undesirable. So for now (if you using my roles (which is probably just me)), I offloaded the task to add zones to Policy to my [glb-zones](https://github.com/ndkprd/ansible-role-fortiadc-dns-zones) role, in which after creating each zones it also add itself to the DNS Policy.

## License

MIT, use at your own risk.