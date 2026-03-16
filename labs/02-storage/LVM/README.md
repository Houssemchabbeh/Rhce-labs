# Lab 06 – Ansible LVM with Error Handling

## Overview

This lab demonstrates how to automate Logical Volume Management (LVM) using Ansible while implementing error handling mechanisms.

The playbook attempts to create a logical volume with a specified size and includes rescue logic if the requested size cannot be allocated.

This lab also demonstrates the use of:

- Ansible blocks
- rescue and always statements
- filesystem creation
- automated mounting

---

## Objectives

- Detect if a Volume Group exists
- Create a Logical Volume using Ansible
- Handle errors using block / rescue
- Implement fallback logic for storage allocation
- Create and format a filesystem
- Configure mount points automatically


---

## Scenario

A system administrator must configure storage automatically using Ansible.

Requirements:

1. Verify if the `research` volume group exists.
2. Attempt to create a logical volume named `data` with a size of **1500MB**.
3. If this fails (for example due to insufficient space), create the logical volume with **800MB** instead.
4. Create a filesystem on the logical volume.
5. Create the required mount point directory.
6. Mount the filesystem automatically if the required disk device exists.

---
### playbook
```yaml
- name: create logical volume and debug errors
  hosts: all
  tasks:
    - name: debug if vg doesnt exist
      block:
        - name: debug 1
          debug:
            msg: "VG not found !"
          when: "'research' not in ansible_lvm.vgs"
        - name: Create a logical volume of 1500M
          community.general.lvol:
            vg: research
            lv: data
            size: 1500m
          when: "'research' in ansible_lvm.vgs"
      rescue:
        - name: debug 2
          debug:
            msg: "could ont create LV of that size"
        - name: Create a logical volume of 800M
          community.general.lvol:
            vg: research
            lv: data
            size: 800m
          when: "'research' in ansible_lvm.vgs"
      always:
        - name: Create a ext3 filesystem
          community.general.filesystem:
            fstype: ext3
            dev: /dev/research/data
        - name: Create a mount point
          ansible.builtin.file:
            path: /part1
            state: directory
            mode: '4755'
        - name: Mount
          ansible.posix.mount:
            path: /mnt/part1
            src: /dev/nvme0n2p1
            fstype: ext3
            state: mounted
          when: '"nvme0n2" in ansible_devices'
```
## Implementation Logic

### 1. Verify Volume Group

The playbook checks whether the `research` Volume Group exists using Ansible facts.

If it does not exist, a debug message is displayed.

---

### 2. Attempt Logical Volume Creation

The automation attempts to create a Logical Volume named `data` with a size of **1500MB**.

---

### 3. Error Handling (Rescue Block)

If the volume creation fails due to insufficient storage space:

- A debug message is displayed
- A fallback logical volume size of **800MB** is created instead

---

### 4. Always Block Operations

Regardless of success or failure, the playbook performs the following tasks:

- Creates an **ext3 filesystem**
- Creates the required mount directory
- Attempts to mount the filesystem




