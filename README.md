#Infrastructure LVM role
This playbook reads the LVM (<http://www.tldp.org/HOWTO/LVM-HOWTO/>) definition from the host variables of an instance and sets up the physical volumes, volume groups and logical volumes accordingly.

General rule for this role: you can only use this playbook to *create* and *extend* disk space in logical volumes. Shrinking and removing disks is **not** supported.


__Requirements__
This playbook cannot be called without a defined `instances` variable. 
If the variable `disks` is not defined in the hostvars of vm, the play will end. 
If `disks` is present, `lvm` is also expected and the play will continue. If `lvm` is not present and `lvm` is, the play will fail.

The minimal version to run this playbook is ANSIBLE 2.8, as some tasks use options that are not availliable in lower versions of ansible. 

__Usage__
In order for this playbook to start making changes, the following items are required:

an `instances` variable (list), filled with hostnames in FQDN, in example:
* node1.domain
* node2.domain

An LVM definition in the host variables (additional disks and pv/vg/lv definitions), in example:
```yml
disks:
- size: 110
  vg: oracle
hostname: ora-hb-1.localdomain
lvm:
  - lv_name: u01
    mountpoint: /u01
    size: '40'
    vg_name: oracle
  - lv_name: oradata
    mountpoint: /u01/app/oracle/oradata
    size: '20'
    vg_name: oracle
  - lv_name: orafra
    mountpoint: /u01/app/oracle/fast_recovery_area
    size: '30'
    vg_name: oracle
  - lv_name: oraredo
    mountpoint: /u01/app/oracle/oraredo
    size: '20'
    vg_name: oracle
```
The definition snippet above will result in:
* one disks of 110GB attached to the virtual machine, all partitioned to contain a single physical volume
* one volume group (oracle on the physical volume)
* four logical volumes that use 100% of the volume group size (oracle_u01 with size 40GB, oracle_oradata with size 20GB, oracle_oraredo with size 20GB and oracle_orafra with size of 30GB)
* A formatted XFS filesystem on all logical volumes
* Creation of mount points on root filesystem
* Modification of /etc/fstab so that the volumes are mounted on boot
* All logical volumes are mounted on the specified mountpoint

The disk size can be:
* 100%: The logical volume will use all available space in the volume group. When this setting is used the volume group **must** contain only **one** logical volume
* an integer: the created logical volume will have a disk size of the provided integer in GB's. So if size is defined as followed: `"size": 2` then the resulting logical volume will have a size of 2GB.

The mountpoint is the location on the root filesystem where the disk will be mounted (in example, /data).
###### Mandatory keys
Both the disks and the lvm keys and their definitions are mandatory for this playbook to work. The minimal working definition consists of a single disk and a single LV on that disk.


__Role Variables__
This playbook does not use default variables.

__Dependencies__
This playbook is not dependant on Ansible Galaxy roles.

__License__
tbd

__Author Information__
Wilco Folkers
