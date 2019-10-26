# filesystems

## 0.3.0

* Refactor `serial` and `sequential` to simplify the code (remove `action.yml`,
  `serial.yml` and `sequential.yml`)
* Rationalize variables names and verify that some of them (those set in task
  `vars` or `loop_var`) are undefined
* Add a fstab validator

## 0.2.0

* Add `serial` and `sequential` behaviours when processing filesystems list
* Add tasks to update paths of Logical Volumes or mountpoints
* Add check points

## 0.1.0

* Add `setup` and `unset` tasks
* Add handlers

## 0.0.1

Init role [`filesystems`](https://github.com/quidame/aqs-role-filesystems)
