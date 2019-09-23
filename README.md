# filesystems

Ansible role to setup or unset filesystems, as well as their underlying Logical
Volumes.

This role is a piece of (*yet another*)
[Ansible Quick Starter](https://github.com/quidame/aqs-common) (**AQS**).

## Requirements

LVM tools must be installed on hosts and a VG must already exist.

## Role Variables

The main action the role is called for. Choices are `setup` (the default) and `unset`.
```yaml
filesystems__action: unset
```

A list of dictionnaries describing filsystems to setup or unset.
```yaml
filesystems__list:
  - path:      **MANDATORY** - No default  - modules: mount, file
    lv:        **MANDATORY** - No default  - modules: lvol (filesystem, mount)
    vg:        Defaults to 'data_vg'   - modules: lvol (filesystem, mount)
    size:      Defaults to '512M'      - modules: lvol
    fstype:    Defaults to 'ext4'      - modules: filesystem, mount
    force:     No default - OMITTED    - modules: lvol, filesystem, file
    resisefs:  No default - OMITTED    - modules: lvol, filesystem
    active:    No default - OMITTED    - modules: lvol
    lvol_opts: No default - OMITTED    - modules: lvol (=opts)
    pvs:       No default - OMITTED    - modules: lvol
    shrink:    No default - OMITTED    - modules: lvol
    mkfs_opts: No default - OMITTED    - modules: filesystem (=opts)
    backup:    No default - OMITTED    - modules: mount
    boot:      No default - OMITTED    - modules: mount
    dump:      No default - OMITTED    - modules: mount
    mountopts: No default - OMITTED    - modules: mount (=opts)
    passno:    No default - OMITTED    - modules: mount
    attr:      No default - OMITTED    - modules: file
    group:     No default - OMITTED    - modules: file
    mode:      No default - OMITTED    - modules: file
    owner:     No default - OMITTED    - modules: file
    selevel:   No default - OMITTED    - modules: file
    serole:    No default - OMITTED    - modules: file
    setype:    No default - OMITTED    - modules: file
    seuser:    No default - OMITTED    - modules: file
```

The following parameters/variables are not implemented:
```yaml
    opts:      Not implemented - modules: lvol, filesystem, mount - CONFLICTS
               See lvol_opts, mkfs_opts and mountopts.
    dev:       Not implemented - modules: filesystem - built from 'vg' and 'lv'
    src:       Not implemented - modules: mount      - built from 'vg' and 'lv'
    state:     Not implemented - modules: lvol, mount, file - HARDCODED
    snapshot:  Not implemented - modules: lvol  - N/A
    thinpool:  Not implemented - modules: lvol  - N/A
    fstab:     Not implemented - modules: mount - N/A
    recurse:   Not implemented - modules: file  - N/A
```

The default values that can be used. Each `filesystems__list`'s dictionnary
key that is not mandatory can be set this way for all filesystems.
```yaml
filesystems__vg: data_vg
filesystems__size: 512M
filesystems__mode: 0755
filesystems__fstype: ext4
```

## Dependencies

None.

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

Setup two filesystems on JBoss servers, with both values specific to the fs
(path, size) and variables valuable for all (mode, owner):
```yaml
- hosts: servers
  roles:
    - role: filesystems
      filesystems__list:
        - path: "{{ jboss_app_dir_path }}"
          size: "{{ jboss_app_vol_size }}"
          lv:   "{{ jboss_app_vol_name }}"
        - path: "{{ jboss_var_dir_path }}"
          size: "{{ jboss_var_vol_size }}"
          lv:   "{{ jboss_var_vol_name }}"
      filesystems__mode: 0750
      filesystems__owner: "webadm"
      filesystems__group: "webgrp"
```

Unset servers:
```yaml
- hosts: servers
  roles:
    - role: filesystems
      filesystems__action: unset
      filesystems__list:
        - path: "{{ jboss_app_dir_path }}"
          lv:   "{{ jboss_app_vol_name }}"
        - path: "{{ jboss_var_dir_path }}"
          lv:   "{{ jboss_var_vol_name }}"
```

## License

GPLv3

## Author Information

<quidame@poivron.org>
