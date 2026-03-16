# Lab – Automated Partition Creation with Ansible

## Overview

This lab demonstrates how to automate disk partitioning using Ansible while implementing error-handling logic.
The playbook attempts to create a partition with a specified size and uses fallback logic if the operation fails.

It also ensures the partition is formatted and mounted automatically.

---

## Objectives

* Verify that a disk device exists on the system
* Create a disk partition using Ansible
* Implement error handling using `block`, `rescue`, and `always`
* Create a filesystem on the partition
* Configure and mount a directory automatically

---


---

## Scenario

A system administrator needs to automate storage configuration on managed hosts.

Requirements:

1. Verify that the disk device `/dev/nvme0n2` exists.
2. Attempt to create a partition of **6 GiB**.
3. If this fails, create a smaller partition of **500 MiB** instead.
4. Create a filesystem on the partition.
5. Create a mount directory.
6. Mount the filesystem automatically.

---

## Storage Architecture

Example structure used in this lab:

```
Disk: /dev/nvme0n2
↓
Partition: /dev/nvme0n2p1
↓
Filesystem: ext4
↓
Mount Point: /mnt/part1
```

---

## Implementation Logic

### 1. Verify Disk Device

The playbook first checks whether the disk device `nvme0n2` exists using Ansible facts.

If the device does not exist, a debug message is displayed.

---

### 2. Attempt Partition Creation

The automation attempts to create a partition of **6GiB** on the disk.

---

### 3. Error Handling (Rescue Block)

If the partition creation fails (for example due to insufficient space):

* A debug message is displayed
* A smaller **500MiB partition** is created instead

---

### 4. Always Block Operations

Regardless of success or failure, the playbook performs the following tasks:

* Creates an **ext4 filesystem** on the partition
* Creates the mount directory
* Mounts the filesystem

These tasks run only if the disk device exists.

---

## Playbook

```yaml
- name: create partition with conditions
  hosts: all
  tasks:
    - name: bloc list
      block:
        - name: ensure 'nvme0n2' exists
          ansible.builtin.debug:
            msg: "'nvme0n2' does not exist"
          when: '"nvme0n2" not in ansible_devices'

        - name: Create partition
          community.general.parted:
            device: /dev/nvme0n2
            number: 1
            state: present
            part_end: 6GiB

      rescue:
        - name: ensure partition created
          ansible.builtin.debug:
            msg: "could not create partition with this size"

        - name: Create a new ext4 primary partition
          community.general.parted:
            device: /dev/nvme0n2
            number: 1
            state: present
            part_end: 500MiB

      always:
        - name: Create a ext2 filesystem on /dev/nvme0n2p1
          community.general.filesystem:
            fstype: ext4
            dev: /dev/nvme0n2p1
            force: true
          when: '"nvme0n2" in ansible_devices'

        - name: create mount folder
          ansible.builtin.file:
            path: /mnt/part1
            state: directory
          when: '"nvme0n2" in ansible_devices'

        - name: Mount
          ansible.posix.mount:
            path: /mnt/part1
            src: /dev/nvme0n2p1
            fstype: ext4
            state: mounted
          when: '"nvme0n2" in ansible_devices'
```

---

## Verification

After executing the playbook, verify that:

* The partition `/dev/nvme0n2p1` exists
* The filesystem was created successfully
* The mount directory `/mnt/part1` exists
* The partition is mounted correctly
use : ansible all -m command -a "lsblk"

---

## Key Concepts Learned

* Automating partition management using Ansible
* Using `block`, `rescue`, and `always` for error handling
* Conditional task execution with `when`
* Filesystem creation and mounting automation

---

## Possible issues during the lab:

* Disk device does not exist on the host
* Insufficient disk space for the requested partition size
* Incorrect device path during filesystem creation
* Mount point not created before mounting

---
## Check if the partition exists & Check mount status
ansible all -m command -a "lsblk"
ansible all -m command -a "mount | grep part1"
