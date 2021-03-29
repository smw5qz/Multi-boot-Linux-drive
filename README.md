# Multi-boot-Linux-drive
Simple process for loading a USB/SD/other drive with multiple bootable Linux distributions

Running Arch Linux, it occured to me that multiple bootable Linux distros on on drive could be quite helpful. When system failure occurs, and cannot be restored, one ought to have a current Arch bootable .iso, and a light, quick rescue OS. I prefer Puppy to get to disk utilities with the target drive unmounted.

This can be done with one partition, with MBR, and a GRUB supported filesystem

Requirements:
-bootable data drive (USB, SD, etc)
-grub

Steps in Terminal, *run sudo su to stay in root or sudo before each line*

1. lsblk # identify device /dev/sdX
2. umount /dev/sdXX #ensure device and all paritions are unmounted 
2. fdisk /dev/sdX
  - o #MBR
  - n #new partition
  - ENTER x 3 # to create partition, Linux part. type by default
  - a #makes it bootable
  - w #writes changes to disk
3. mkfs.vfat -F32 /dev/sdX1 #formats the new partition to FAT32
4. mkdir /mnt/boot #creates mount point
5. mount /dev/sdX1 /mnt
6. cd to /mnt/boot, then mkdir iso #folder for the isos
7. load your Linux .iso(s) into the new iso folder
8. grub-install --target=x86_64-efi --removable --boot-directory=/mnt/boot --efi #does not require your devpath
9. grub-mkconfig /mnt/boot/grub/grub.cfg #creates grub config on the drive based on your system's configO
