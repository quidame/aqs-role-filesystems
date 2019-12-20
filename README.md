# filesystems

Ansible role to setup or unset filesystems, as well as their underlying Logical
Volumes, their mountpoints and their parent directories.

This role is a piece of (*yet another*) [Ansible Quick Starter](/aqs-common)
(**AQS**).

## Requirements

LVM tools must be installed on hosts and a VG must already exist. Also,
non-default FS tools must be installed apart to be able to use these specific
filesystem types. This role installs no package.

## Role Variables

The wanted state of the LV/FS the role is played for. Choices are `present` (the
default), `absent` or `unmounted`.
```yaml
filesystems__state: absent
```

A list of dictionaries describing filsystems to setup or unset.
```yaml
filesystems__fslist:
  - path: /opt/foobar
    lv: foobar_app
  - path: /var/foobar/cache
    size: 4G
    lv: foobar_var
```

| Key | Mandatory | Default | Description |
|:----|:---------:|:--------|:------------|
| `path` | **always** | | Absolute path of the mountpoint |
| `lv` | **always** | the one already mounted on `path`|Name of the Logical Volume |
| `vg` | **always** | `vg0` | Name of the VG the LV belongs to |
| `size` | *present* | `512M`| Size of the Logical Volume |
| `fstype` | *present* | `ext4` | Filesystem type to mount on `path` |
| `force` | no | `false` | Shortcut to force almost all operations - not recommended |
| `force_mkfs` | no | same as `force` | Force `filesystem` module |
| `force_kill` | no | same as `force` | Send SIGKILL (`true`) or SIGTERM (`false`) to processes accessing the FS |
| `force_wipe` | no | same as `force` | Force wipefs command on mounted filesystems |
| `force_lvol` | no | dynamically set | Force `lvol` module while creating/resizing the LV |
| `resizefs` | no | dynamically set | Allow `lvol` module (not `filesystem`) to resize FS |
| `shrink` | no | dynamically set | Allow `lvol` module to reduce the LV |

Also, almost all parameters of the `file` module have a matching key to set
access permissions at the root of the FS.

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
      filesystems__state: present
      filesystems__fslist:
        - path: "{{ jboss_app_dir_path }}"
          size: "{{ jboss_app_vol_size }}"
          lv:   "{{ jboss_app_vol_name }}"
        - path: "{{ jboss_var_dir_path }}"
          size: "{{ jboss_var_vol_size }}"
          lv:   "{{ jboss_var_vol_name }}"
      filesystems__default_mode: "0750"
      filesystems__default_owner: "webadm"
      filesystems__default_group: "webgrp"
```

Unset servers:
```yaml
- hosts: servers
  roles:
    - role: filesystems
      filesystems__state: absent
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
