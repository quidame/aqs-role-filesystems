# filesystems

Ansible role to setup or unset filesystems, as well as their underlying Logical
Volumes, their mountpoints and their parent directories.

This role is a piece of (*yet another*) [Ansible Quick Starter](/aqs-common)
(**AQS**).

## Requirements

LVM tools must be installed on hosts and a VG must already exist. Also, non-
default FS tools must be installed apart to be able to use specific filesystem
types.

## Role Variables

The main action the role is called for. Choices are `setup` (the default) and
`unset`.
```yaml
filesystems__action: unset
```

A list of dictionnaries describing filsystems to setup or unset.
```yaml
filesystems__fslist:
  - path:      MANDATORY - No default  - modules: mount, file
    lv:        MANDATORY - No default  - modules: lvol (filesystem, mount)
    vg:        Defaults to 'vg0'       - modules: lvol (filesystem, mount)
    size:      Defaults to '512M'      - modules: lvol
    fstype:    Defaults to 'ext4'      - modules: filesystem, mount
    force:     No default - OMITTED    - modules: lvol, filesystem, file
    resisefs:  No default - OMITTED    - modules: lvol, filesystem
    lvol_opts: No default - OMITTED    - modules: lvol (=opts)
    pvs:       No default - OMITTED    - modules: lvol
    shrink:    No default - OMITTED    - modules: lvol
    mkfs_opts: No default - OMITTED    - modules: filesystem (=opts)
    backup:    No default - OMITTED    - modules: mount
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

The following keys, matching other modules parameters by their names, are not
implemented here and, for almost all, are not applicable:
```yaml
    opts:      Not implemented - modules: lvol, filesystem, mount - CONFLICTS
               See lvol_opts, mkfs_opts and mountopts.
    dev:       Not implemented - modules: filesystem - built from 'vg' and 'lv'
    src:       Not implemented - modules: mount      - built from 'vg' and 'lv'
    state:     Not implemented - modules: lvol, mount, file - HARDCODED
    active:    Not implemented - modules: lvol  - N/A
    snapshot:  Not implemented - modules: lvol  - N/A
    thinpool:  Not implemented - modules: lvol  - N/A
    boot:      Not implemented - modules: mount - N/A
    fstab:     Not implemented - modules: mount - N/A
    recurse:   Not implemented - modules: file  - N/A
```

The default values that can be used. Each `filesystems__list`'s dictionnary
key that is not mandatory can be set this way for all filesystems, and can
be set/overridden on a per-fs basis.

The default VG the LV belongs to.
```yaml
filesystems__default_vg: vg0
```

The default size of the LV to create, shrink or extend.
```yaml
filesystems__default_size: 512M
```

The default filesystem type.
```yaml
filesystems__default_fstype: ext4
```

Other default parameters that are not set by the role and so get the default
values of the related modules.
```yaml
filesystems__default_force:     (no)
filesystems__default_resisefs:  (no)
filesystems__default_lvol_opts: ()
filesystems__default_pvs:       ()
filesystems__default_shrink:    (yes) needs force=yes to be applied
filesystems__default_mkfs_opts: ()
filesystems__default_backup:    (no)
filesystems__default_mountopts: ()
filesystems__default_dump:      (0)
filesystems__default_passno:    (0)
filesystems__default_attr:      ()
filesystems__default_mode:      () results may depend on umask
filesystems__default_group:     (root)
filesystems__default_owner:     (root)
filesystems__default_selevel:   (s0)
filesystems__default_serole:    ()
filesystems__default_setype:    ()
filesystems__default_seuser:    ()
```

The 3 following variables are set in `vars/main.yml` to not be overridden from
inventory.

This variable defines the behaviour of the role when processing the filesystems
list: `serial` (the default) or `sequential`.
```yaml
filesystems__behaviour: sequential
```

This variable defines if yes or no changes in Logical Volume names or paths of
their mountpoints have to be updated. This renaming feature is enabled by
default and can be disabled for debugging purposes.
```yaml
filesystems__update_paths: False
```

Almost all changes will trigger a handler to validate contents of `/etc/fstab`
system file. This handler is a series of tasks to:
- assert there is no duplicates in the file. Are considered as duplicated
  devices paths such as `/dev/foo/bar` and `/dev/mapper/foo-bar`, and as
  duplicated mountpoints paths such as `/srv/foobar` and `/srv/foobar/`.
- assert there is no unpredictable name (as `/dev/sdb2` or `/dev/dm-6`) in use
  in the file.
- assert that fstypes recorded in fstab match the ones in `ansible_mounts` for
  a same mountpoint, if mounted.
- check that all recorded devices and mountpoints exist.
- finish to validate fstab entries by performing acual mount calls, with option
  `-o remount` for mounted filesystems (this will validate mount options) and
  with no option at all for unmounted ones (this will validate mount options as
  well as fstype.

This handler can be skipped with:
```yaml
filesystems__validate: False
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

Setup two filesystems on JBoss servers, with both values specific to the fs
(path, size) and variables valuable for all (mode, owner):
```yaml
- hosts: servers
  roles:
    - role: filesystems
      filesystems__action: unset
      filesystems__fslist:
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
      filesystems__fslist:
        - path: "{{ jboss_app_dir_path }}"
          lv:   "{{ jboss_app_vol_name }}"
        - path: "{{ jboss_var_dir_path }}"
          lv:   "{{ jboss_var_vol_name }}"
```

Validate `/etc/fstab`:
```yaml
- hosts: servers
  tasks:
    - include_role:
        name: filesystems
	tasks_from: validate-fstab.yml
```

## License

GPLv3

## Author Information

<quidame@poivron.org>
