# filesystems

Ansible role to setup or unset filesystems.

A brief description of the role goes here.

This role is a piece of (*yet another*)
[Ansible Quick Starter](https://github.com/quidame/aqs-common) (**AQS**).

## Requirements

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

The main action the role is called for. Choices are `setup` (the default) and `unset`.
```yaml
filesystems__action: unset
```

## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Installation

To make use of this role has a galaxy role, put this in `requirements.yml`:

```yaml
- name: "filesystems"
  src: "https://github.com/quidame/aqs-role-filesystems.git"
  scm: "git"
  version: master
```

And then run:

```bash
ansible-galaxy install -r requirements.yml
```

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

Setup servers:
```yaml
- hosts: servers
  roles:
     - role: filesystems
```

Unset servers:
```yaml
- hosts: servers
  roles:
     - role: filesystems
       filesystems__action: unset
```

## License

GPLv3

## Author Information

<quidame@poivron.org>