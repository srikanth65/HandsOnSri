
**Extend the file system after resizing an Amazon EBS volume**

Before you Begin: 
- Create a snapshot of the volume, in case you need to roll back your changes.
- Confirm that the volume modification succeeded and that it is in theoptimizing or completed state
- Ensure that the volume is attached to the instance and that it is formatted and mounted
- (Linux instances only) If you are using logical volumes on the Amazon EBS volume, you must use Logical Volume Manager (LVM) to extend the logical volume. 

**Step1:** After you increase the size of an EBS volume, you must extend the partition and file system to the new, larger size. You can do this as soon as the volume enters the optimizing state.

**Step2**

**Extend the file system of EBS volumes**

  - To extend the file system of EBS volumes
      - 1. Connect to your instance.
      - 2. Resize the partition, if needed. To do so: sudo lsblk
      - 3. Get the name, size, type, and mount point for the file system that you need to extend.
           ```
           df -hT or lsblk -f
           sudo file -s /dev/xvdb
           sudo mkfs -t xfs /dev/xvdb
           mkdir newVolume
           mount /dev/xvdb newVolume/
           mount -all
           ```
      - 4. sudo xfs_growfs -d /dev/xvdb        # Use the xfs_growfs command and specify the mount point of the file system
      - 5. sudo growpart /dev/xvda 1         # Extend the partition. Use the growpart command and specify the device name and the partition number.

Check whether the volume has a partition. Use the lsblk command.
