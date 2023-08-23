# Increasing VM Harddrive size

### Expanding Harddrive size in VMWare Workstation

1. Power off VM
2. Select VM -> Settings
3. Under Hardware select Hard Disk -> Expand -> Expand to desired size
4. Boot VM on and run as root
5. Take snapshot (just in case)
6. Identify device name (by default /dev/sda) and confirm the new size
   1. `fdisk -l` <img src="../../.gitbook/assets/image (169) (1).png" alt="" data-size="original">
7. Create a new primary partition:
   1. fdisk /dev/sda (depending on device name)
   2. Press p to print the partition table to identify the number of partitions.&#x20;
   3. Press n to create a new primary partition.
   4. Press p for primary.
   5. Press 3 for the partition number (dev/sda3 is going to be the new partition)
   6. Press Enter two times.
   7. Press t to change the system's partition ID.
   8. Press 3 to select the newly created partition.
   9. Type 8e to change the Hex Code of the partition for Linux LVM.
   10. Press w to write the changes to the partition table.
8. Restart the virtual machine.
9. Verify that the changes were saved to the partition table
   1. `fdisk -l`
10. Convert the new partition to a physical volume
    1. `pvcreate /dev/sda3`
11. Determine which volume group to extend

    1. `vgdisplay`

    ![](<../../.gitbook/assets/image (171) (1).png>)
12. Extend the physical volume
    1. `vgextend kali-vg /dev/sda3`
13. Verify how many physical extents are available
    1. `vgdisplay kali-vg| grep "Free"`
14. Determine which logical volume

    1. `lvdisplay`

    ![](<../../.gitbook/assets/image (170) (1).png>)
15. Extend the Logical Volume
    1. `lvextend -L+#G /dev/kali-vg/root`
    2. \# is the number of free space in GB
16. Expand the ext3 filesystem online
    1. `resize2fs /dev/kali-vg/root`
    2. Use resize2fs instead of ext2online for non-Red Hat virtual machines.
    3. Use xfs\_growfs for Red Hat, CentOS 7 and other VM Guest OS types that use the XFS file system
17. Verify that the filesystem has the new space
    1. &#x20;`df -h`&#x20;
