# Day 13 – Linux Volume Management (LVM)

# Switch to Root User

```bash
sudo -i
Create Virtual Disk
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024
Attach Loop Device
losetup -fP /tmp/disk1.img
Verify Loop Device
losetup -a

Observed loop device like /dev/loop0

Task 1: Check Current Storage
lsblk
pvs
vgs
lvs
df -h

Observation:
Checked available disks, physical volumes, volume groups, logical volumes, and mounted storage.

Task 2: Create Physical Volume
pvcreate /dev/loop0
Verify Physical Volume
pvs

Observation:
Successfully created physical volume.

Task 3: Create Volume Group
vgcreate devops-vg /dev/loop0
Verify Volume Group
vgs

Observation:
Volume group devops-vg created successfully.

Task 4: Create Logical Volume
lvcreate -L 500M -n app-data devops-vg
Verify Logical Volume
lvs

Observation:
Logical volume app-data created successfully.

Task 5: Format and Mount
Format with ext4
mkfs.ext4 /dev/devops-vg/app-data
Create Mount Directory
mkdir -p /mnt/app-data
Mount Logical Volume
mount /dev/devops-vg/app-data /mnt/app-data
Verify Mounted Volume
df -h /mnt/app-data

Observation:
Logical volume mounted successfully.

Task 6: Extend the Volume
Extend Logical Volume
lvextend -L +200M /dev/devops-vg/app-data
Resize Filesystem
resize2fs /dev/devops-vg/app-data
Verify Extended Size
df -h /mnt/app-data

Observation:
Logical volume size increased successfully.

Commands Used
lsblk
pvs
vgs
lvs
pvcreate
vgcreate
lvcreate
mkfs.ext4
mount
lvextend
resize2fs
df -h
What I Learned
LVM provides flexible storage management in Linux
Physical Volumes, Volume Groups, and Logical Volumes work together
Logical volumes can be extended without recreating disks
Mounting makes storage accessible to the system
LVM is important for scalable DevOps infrastructure