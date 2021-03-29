# Multi-boot-Linux-drive
Simple process for loading a USB/SD/other drive with multiple bootable Linux distributions
Clarifies this wiki: https://wiki.archlinux.org/index.php/Multiboot_USB_drive

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
4. mlabel -i /dev/sdX1 ::NAME_YOUR_DRIVE #labels the drives
5. mkdir /mnt/boot #creates mount point
6. mount /dev/sdX1 /mnt
7. cd to /mnt/boot, then mkdir iso #folder for the isos
8. load your Linux .iso(s) into the new iso folder
9. grub-install --target=x86_64-efi --removable --boot-directory=/mnt/boot --efi #does not require your devpath
10. grub-mkconfig /mnt/boot/grub/grub.cfg #creates grub config on the drive based on your system's config

#NOTE: At this point we must tread carefully to ensure each iso shows up on the GRUB menu  
11. Edit the grub.cfg, keep your 00 initalization section, clear the 10, 20, 30, 40 and beyond
12. IMPORTANT: change the both timeouts in the code to 10, otherwise first menu items will be auto-selected
13. Put this at the start of your 10_Linux section, sets the path to your labels device, which is persistent. Devpaths are not across sessions.
set imgdevpath="/dev/disk/by-label/YOURLABELNAME"

14. Set your first GRUB menu entry, below is a working example for ArchLinux
menuentry '[loopback]archlinux-2020.10.01-x86_64.iso' {
	set isofile='/boot/iso/archlinux-2020.10.01-x86_64.iso'
	loopback loop $isofile
	linux (loop)/arch/boot/x86_64/vmlinuz-linux img_dev=$imgdevpath img_loop=$isofile earlymodules=loop
	initrd (loop)/arch/boot/intel-ucode.img (loop)/arch/boot/amd-ucode.img (loop)/arch/boot/x86_64/initramfs-linux.img
}

EXAMPLE for a different distro, Puppy Linux. Adapted from the template above
menuentry 'Puppy Linux Live' {
        set isofile='/boot/iso/pup.iso'
        loopback loop $isofile
        linux (loop)/vmlinuz img_dev=$imgdevpath img_loop=$isofile pfix=copy,fsck pmedia=cd
        initrd (loop)/initrd.gz
}

15. Save the config file, reload, boot from the drive and enjoy!

Note: These examples are one menu entry for a distribution. Arch and Puppy's live menu option. Other desired options need to be added.
Open the iso of choice to study its /boot/grub/grub.cfg, and add the menu items you want.
