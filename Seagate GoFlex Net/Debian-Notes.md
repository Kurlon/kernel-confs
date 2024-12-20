Random notes from attempting to install Debian 13 (testing) from Gentoo on a GoFlex Net.

- Prep the target filesystem. Two partitions, rootfs and swap.  Use e2label to label / rootfs
   - U-Boot 2016 can boot off ext4, but doesn't support 64bit features, use 'mkfs.ext4 -O ^metadata_csum,^64bit /dev/sdX1' when formatting.
   - https://forum.doozan.com/read.php?3,12381 has updated U-Boot images, may have a fix for this?
   - mount rootfs somewhere like /mnt/debian

 - Download debootstrap https://ftp.debian.org/debian/pool/main/d/debootstrap/
   - Use deb2targz on Gentoo to extract it.
   - cp the usr/share/debootstrap dir to /usr/share/debootstrap
   - call it via usr/sbin/debootstrap --arch armel testing /mnt/debian https://deb.debian.org/debian

 - Chroot in, configure using this as a rough guide: https://gist.github.com/varqox/42e213b6b2dde2b636ef#set-up-filesystem-for-debian
   - Before installing the kernel, update sources.list
     - When setting up sources, add non-free AND non-free-firmware
   - Install u-boot-tools initramfs-tools
   - For the kernel, 13 only has RPi stuff at the moment, get a kernel with install instructions at https://forum.doozan.com/read.php?2,12096
     - Install firmware-linux THEN process the initrd
