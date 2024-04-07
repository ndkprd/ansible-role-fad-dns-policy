# ansible-role-fortiadc-dns-policy

## Description

Ansible role to create/update Fortinet's FortiADC Global DNS Policy via their HTTP REST API. 

## Prerequisite

The DNS Policy API need 3 other resources to be exists first: *Address Group*, *Remote DNS Servers*, and *Response Rate Limit*. At this point in time I haven't created the roles to create/update them, so if you want to use customized resource for those, you need to make sure your customized resources entry you put in variable is already exists.

For the 

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
    fad_vdom: root

  roles:
    - ndkprd.fortiadc-glb-vsp

```

### About Tags

I added quiet lots of debug task, mainly to check if the variable I set is correct. These tags basically just print out the var that the previous task set/register. You can skip them altogether by skipping tasks with `debug` tags.

For example, if you're using CLI, you can just go `ansible-playbook playbook.yaml --skip-tags debug`.

## Limitation

Developed and tested against FortiADC 7.0.

## License

MIT, use at your own risk.