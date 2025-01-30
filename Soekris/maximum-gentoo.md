Maximum Gentoo - Full send for a 486
- Custom OpenRC stage 3 built for 486 with -Oz and musl, maybe LTO?

Step 0 - Soekris 4501 setup
 - PXE Booting to start, just requires serial console access and DHCP to be setup properly.  PXELinux and IPXE (old version, build on Ubuntu 14.04 or equivilent) both work. Will also need a tftp host to hold kernel, initramfs, and bootloaders.
   
Step 1 - Gentoo build farm
- HP DL160 G6
  - 2 x Xeon X5650 (12 cores total at 2.66GHz)
  - 96GB DDR3 @ 1333MHz
- HP DL150 G6 (Was going to be used as additional distcc horsepower, skipped it for simplicity on first try.)
  - 2 x Xeon E5645 (12 cores total at 2.4Ghz)
  - 96GB DDR3 @ 1333MHz
- Booster - NBD host
  - Gentoo VM used for light distcc duty, and now hosting NBD servers for my 486 'cause it's limited to CF storage currently, and all I have working is a 256MB card.

Step 2 - Start the Initial chroot - https://wiki.gentoo.org/wiki/Handbook:X86/Full/Installation
- Go through the x86 Handbook, using the NBD lun to hold the env. I'm doing a normal MBR layout, boot, root, swap, mounting to /mnt/gentoo, arch-chroot, etc. Use the i686-musl stage 3 snap to base off of.
- Do an emerge-webrsync.
- Change gears, switch to the Changing CHOST instructions. - https://wiki.gentoo.org/wiki/Changing_the_CHOST_variable
  - New CHOST will be i486-pc-linux-musl
  - New CHOST_x86 will match CHOST
  - New CFLAGS will be "-O2 -march=i486 -pipe"
  - When doing the /etc/env.d check, you're grepping for linux-musl, not linux-gnu like the example shows.
  - Once the full @world build completes, you can safely go ham on CFLAGS if desired and rebuild again to implement.
  - Back to the handbook from here...

Step 3 - Kernel / boot stuffs
- Build installkernel, setting it up for grub and dracut
  - Grub will be used to switch from pxeboot to local boot once the system is live.
- Install sys-block/nbd
- Install net-misc/dhcp to get dhclient which allows dracut to use the network-legacy module
- Update dracut's conf to be hostonly, use nbd, network-legacy and base modules only
- Install Gentoo-sources, use stripped down kernel conf

GCC Option Comparison (-O2 vs -Os vs -Oz)
Note this is only looking at binary size, performance isn't considered / benched yet. It's also assumed that the overall memory footprint of a given binary isn't changed, IE going from -O3 to -Oz isn't going to prevent xz from eating 60MB of working ram to do it's job, just the binary's size in RAM will hopefully shrink. The theory here is on VERY RAM constrained systems, the loss of raw perf by going to a lower optimization level will be offset by not paging to swap as often by virtue of executable code being smaller.

I'm testing this by doing a full world rebuild in a chroot, just changing cflags to see what happens at the macro level. No performance testing has been done to compare the speed impact of these tweaks. I'm breaking out -fomit-frame-pointer as it only kicks in on -march=i486 at O2 and up.
			
| GCC Options	| bzImage	| /usr/lib | /usr/bin | /usr/libexec |
| --- | --- | --- | --- | --- |
| -O2 | 4336 | 403928 | 108320 | 182564 |
| -O2 -flto | | 401096 | 98044 |192632 |
| -O2 -flto -ffunction-sections -fdata-sections -Wl,--gc-sections | | 400900 | 94880 | 191936 |
| -Os | 5248 | 384272 | 73088 | 182044 |
| -Oz | | 282596 | 72549 | 182028 |
| -Oz -fomit-frame-pointer -flto -ffunction-sections -fdata-sections -Wl,--gc-sections | | 385590 | 67348 | 191744 |
| -Os -flto -ffunction-sections -fdata-sections -Wl,--gc-sections | | 382192 | 64056 | 191620 |
| -Oz -flto -ffunction-sections -fdata-sections -Wl,--gc-sections | | 380532 | 63620 | 191628 |
