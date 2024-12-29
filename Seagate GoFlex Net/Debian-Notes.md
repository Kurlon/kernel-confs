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
   - Install u-boot-tools initramfs-tools bzip2 xz-utils man-db wget
   - For the kernel, 13 only has RPi stuff at the moment, get a kernel with install instructions at https://forum.doozan.com/read.php?2,12096
     - Install firmware-linux THEN process the initrd
   - Setup /etc/fw_env.config:
     - /dev/mtd0 0xc0000 0x20000 0x20000

- Random fun info:
  - https://www.kernel.org/doc/Documentation/devicetree/bindings/leds/common.yaml
    - GPIO LEDs can now have automated triggers in kernel instead of via scripts, disk / network activity, system load, etc all handled in kernel.

- Dumb maximum fast build - don't do this!
  - Prep the new volume, rootfs will be ext4, typically first partition. Swap as a second partition, no reason to go too big as this box falls over hard at about 300MB swapped. Aim for 2GB total (will have two swap partitions in production.) as the box will be OOM killing processes well before you get there.
    - mkfs.ext4 -E discard -L rootfs -O ^metadata_csum,^64bit /dev/blah
      - This 'trims' the space for SSDs (could just secure erase first...), and says we don't need no checksums, or big volumes in an attempt to keep IOs small and fast. This is also slightly stupid and will have consequences in the case of a crash... You have backups, right? As a bonus, older u-boot can read this without issue.
    - mkswap /dev/blah
  - RootFS prep, mount a new volume to /mnt/debian on an existing Debian ARMEL system so you can chroot/etc all from one spot.
    - mount -o nobarrier,data=writeback /dev/blah /mnt/debian
      - uEnv.txt for this install will need to be told to use the updated data mode with "root-flags=data=writeback" added. Not bothering to attach swap 'cause you've already got enough on your existing system, right?
    - debootstrap --arch armel testing /mnt/debian https://deb.debian.org/debian
    - Follow guide for chroot as above with the following notes:
      - Install vim-doc when installing vim.
      - Setting up fstab
        - Rootfs mount options: defaults,relatime,discard,nobarrier,data=writeback
        - swap mount options: sw,discard=once,pri=0
      - sources.list:
        - deb https://deb.debian.org/debian testing main non-free non-free-firmware
        - deb-src https://deb.debian.org/debian testing main contrib non-free non-free-firmware
      - After updating sources, apt install u-boot-tools initramfs-tools bzip2 bzip2-doc xz-utils man-db wget firmware-linux bash-completion systemd-timesyncd openssh-server
        - Consider swapping in dropbear for openssh-server if also skipping Network-Manager.
      - Install locally built tweaked bodhi kernel with DisplayLink support, setup boot as per bodhi's instructions.
        - Shrink initramfs, set tmpdir size to 32M
      - Setup /boot/uEnv.txt:
        - custom_params=mitigations=off plymouth.enable=0 disablehooks=plymouth root-flags=data=writeback
      - You've already installed firmware, no need to repeat that step
      - Network-manager is cute and all... but it needs dbus and other noise (as does openssh-server so, not as big a savings as hoped)... systemd-networkd is right there. It doesn't support wireless without additional help, so keep that in mind. Pick a path, set it up now or deal with it post install via serial console.
      - Don't install a bootloader, you've already got one at home. OpenSSH-Server was also already installed earlier.
      - useradd UNAME -m -s /bin/bash
        - Why they default to /bin/sh I don't know, and I get grumpy when I forgot to fix it.
      - Do NOT use tasksel, particularly to install a desktop env. Doing so triggers something to be added that causes nearly every usb network / bluetooth / wireless kernel module to be loaded at boot for some reason. Additionally, even lxde as fully setup by tasksel is just too fat for a 128MB RAM box. Xorg with a window manager will have to be more handrolled to maximize usefullness on the GoFlex.
      - Console setup, os prober, etc, skip the rest.
      - exit, umount -R /mnt/debian
   - Reboot time, poweroff, shuffle drives as needed, boot.
