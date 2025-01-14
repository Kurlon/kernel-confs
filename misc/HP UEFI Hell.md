- Assumptions:

  - SecureBoot is desired and enabled. Windows 11 is in use, it REALLY wants SecureBoot, no point fighting it, let’s play in the pool nicely.

  - Windows 11 is currently installed and working.

  - The target computer is an HP with a maliciously compliant _‘open’_ UEFI implementation. For my testing I’m using an HP Probook 450 G2 on the latest BIOS released.

    - Will only look for Microsoft bootloaders on hard drives or nvme devices.

    - Ignores boot order config changes from other OSs, and sometimes even throws errors when they try.

    - When FastBoot is enabled in the BIOS it won’t even attempt booting off other media, just the internal drive, and only a Microsoft bootloader.

  - Linux will be installed on a separate drive. For the doc I’m assuming Mint but the theory should apply to any flavor distro. Make sure the distro you choose works with SecureBoot, both after the install and via the install media! For Mint both are only true with 21.3 and later.

  - [RUFUS](https://rufus.ie/en/) or a similar tool that can flash ISOs to bootsticks as a filesystem rather than just DDing the image down raw is being used to make the Linux installer thumbdrive.

    - [Ventoy](https://www.ventoy.net/en/index.html) can be used as well, this requires an extra step to get booting happily without disabling SecureBoot: <https://www.ventoy.net/en/doc_secure.html>

  - You’re reasonably comfortable in Linux, including the risks of doing things as root and being able to identify which block device is what on your system.

- Prep - HP BIOS Settings (F10 at power up):

  - `SecureBoot` enabled, this will force the boot mode to `UEFI without CSM`.

  - Disable `FastBoot` to allow booting USB without pressing F9 to bring up the alternate boot menu.

  - Alter the boot order, move `USB Disk` and `Generic USB Device` to be the first two items in the list.

  - You can also change the timeout from 0 to 5 or higher on `Multiboot Express Popup Delay` to always see the alternate boot menu at power up. As you may still need to manually select the right boot file for Mint, it’s a good idea to turn this on to start.

- Prep - Boot Media:

  - If using Ventoy - Enroll Ventoy’s key in MOK to allow it to boot:

    - <https://www.ventoy.net/en/doc_secure.html>

  - If using Mint 21.3 or later ISO directly - Use Rufus or similar:

    - GPT Partition format (UEFI boot only)

    - Write in DD mode, not ISO

    - I’ve had better luck with small (8GB, etc) sticks vs larger? My 128GB stick the BIOS will not see as a boot option. My 8GB stick written in DD mode boots cleanly even with secure boot lit.\
      \
      To test, power up the machine with your boot stick plugged in, hit F9 for the alternate boot menu and see if USB Media shows up as a boot option. If it doesn’t, try other ports, or a different thumbstick. Make note of which USB ports work.

- Prep - Second Boot Stick:

  - To work around HP’s horrible UEFI limitations, this process utilizes a dedicated thumbstick for booting Linux once installed. This thumbstick will get it’s own EFI partition put on it, along with /boot. When you want to boot into Windows, boot without the thumbstick plugged in. To boot into Linux, boot with it plugged in. Keeping the EFI partitions separate prevents Windows from stomping on Linux’s boot setup, and keeps Linux from mangling Windows. A 2GB or larger stick should be PLENTY.\
    \
    The same limitations apply to it as the Linux install media, it has to be a device that UEFI will accept as bootable so test it by putting the Mint install on it and test the alternate boot menu to verify it shows up too.

  - Label this drive so you don’t mistake it for anything else.

- Installing Linux Mint

  - Plug in your Linux boot method of choice, reboot into Mint installer. (F9 at power up.) Remember that not all USB ports will be bootable, you may need to try multiple ones to find ones that HP sanctions for boot. Label your Mint Boot Stick so it’s easy to identify.

    - Once in the live env, bring up a terminal and sudo su - to root.

      - Your Windows disk needs to be identified and ‘hidden’ from Linux so the installer won’t try to monkey with it’s EFI boot partition. You’re root, GO SLOW because you can cause damage from this point forward.

      - **ls /sys/block** should show all active drives, such as sda, sdb, nvme0, etc. One of these will be your Mint install stick, one will be your Windows drive, and one should be your planned Linux drive. You can use fdisk or other tools to look at the partition maps to help identify drives. In this example the Windows drive is `sda`.

      - **echo 1 > /sys/block/**_sda_**/device/delete** will trigger Linux to drop the device so it won’t be seen or touched by the installer.

    - Plug in your dedicated linux boot thumbstick.

      - There is a chance your boot thumbstick will pick up the same block device ID as the drive you dropped, so reviewing the available block devices again after plugging it in is a good idea. 

  - Start the Installer

    - At the `Multimeda codecs` choice screen, enable them, check the `Configure SecureBoot` box and put in a password. Put this in a PW Safe on a phone or some other secure storage spot. This is to allow third-party drivers (nVidia) to play nice. If you forget this, you’re going to have a bad time. Keep this simple, short, and something you can easily remember, for example the 4th letter of it is on demand. (Not all MOK utils are this annoying, mine isn’t, but some are.) This will make sense when you see the first reboot instructions. Also, keep using the same password if you make multiple attempts, the keys possibly can’t be removed from your system if you forget the associated PW.

    - For `Installation type` choose `Something else`, if you see an option to install beside Windows, something’s wrong and the installer can see your Windows drive and EFI partition, STOP HERE.

      - Find your chosen boot stick in the drive layout tool, setup a new partition table. In the new free space, add a new partition, type `EFI System Partition`. This only needs to be 256MB or so. Note what it is, `/dev/sdc1` for example.

      - Use the rest of the space on the boot stick for a second partition, EXT4 mounted at `/boot`.

      - Set the `Device for boot loader installation` to `/dev/sdc` or whichever drive you opted to put the new EFI partition on, possibly already selected by the installer but double check.

      - Add a root partition at a minimum, /home and or swap if you want them separate, etc to the dedicated Linux drive.

    - Finish the install, reboot.

      - When prompted, only unplug the Linux Mint installer thumbstick, leave the new Boot thumbstick in place.

      - The BIOS will initially not accept the new bootloader’s signature for Linux and will bring up the blue MOK management screen. You only get ONE chance to do this, otherwise you’re using the installer again to do some mokutil voodoo.

        - Choose Enroll MOK

        - Choose Continue to work with key 0

        - Choose Yes to Enroll the key.

        - Enroll, enter the secure boot password you setup earlier, you won’t get any feedback while typing.

        - Choose Reboot.

      - If your machine boot loops with the boot thumbstick in place, use the alt boot menu (F9) and choose to boot from a file.

        - Select the USB stick

        - Browse to `/EFI/ubuntu` and select `shimx64.efi` to boot.

- If Windows won’t boot:

  - F9, `Boot from EFI File` on your primary disk:

    - `EFI\Microsoft\Boot\bootmgrfw.efi`

      

- ToDo:

  - udev rule to hide the Windows drive from Mint post install to prevent cross contamination.

  - Linux config to account for Windows storing local time in the RTC vs UTC like Linux prefers. Mint normally should autodetect this, but as we’re hiding Windows from it that may not be the case.

    - **sudo timedatectl set-local-rtc 1**

    - Arch Linux suggests setting Windows to store UTC instead in [their guide](https://wiki.archlinux.org/title/System_time#UTC_in_Microsoft_Windows) for fewer overall headaches.

  - mokutil shenanigans recovery

    - Having a working Ventoy stick is handy, if you use mokutil it expects there to be a working mmx64.efi on whatever efi boot vol it randomly decrees is active. If that isn’t present on your boot media, you’ll get stuck in a loop of not being able to boot anything BUT windows until you go through mok setup successfully, which requires a boot stick with mm64.efi. Ventoy satisfies this requirement. Stock Mint install sticks do not.

  - Can ‘customized boot’ eliminate the need to select the boot file every time? Or maybe efiutils?

  -
