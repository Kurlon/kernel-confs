Maximum Gentoo - Full send for a 486
- Custom OpenRC stage 3 built for 486 with -Oz and musl, maybe LTO?

Step 1 - Gentoo build farm
- HP DL160 G6
  - 2 x Xeon X5650 (12 cores total at 2.66GHz)
  - 96GB DDR3 @ 1333MHz
- HP DL150 G6 (Was going to be used as additional distcc horsepower, skipped it for simplicity on first try.)
  - 2 x Xeon E5645 (12 cores total at 2.4Ghz)
  - 96GB DDR3 @ 1333MHz
- Emachine - iSCSI host
  - OmniOS lab setup, easy and quick for me to farm out an iSCSI lun on for testing. I'm using a Soekris 4501 as my 486, plan is to PXE boot it off iSCSI as I don't have any suitable CF cards that work in it at the moment.

Step 2 - Initial chroot - https://wiki.gentoo.org/wiki/Handbook:X86/Full/Installation
- Go through the x86 Handbook, using the iSCSI lun to hold the env. I'm doing a normal MBR layout, boot, root, swap, mounting to /mnt/gentoo, arch-chroot, etc. Use the i686-musl stage 3 snap to base off of.
- Do an emerge-webrsync.
- Change gears, switch to the Changing CHOST instructions. - https://wiki.gentoo.org/wiki/Changing_the_CHOST_variable
  - New CHOST will be i486-pc-linux-musl
  - New CHOST_x86 will match CHOST
  - New CFLAGS will be "-Oz -march=i486 -pipe"
  - When doing the /etc/env.d check, you're grepping for just linux, not linux-gnu like the example shows.
  - Setup make.conf to use -j24 -l25, add --jobs 4 to the full world rebuild emerge to go moar faster.

GCC Option Comparison (-O2 vs -Os vs -Oz)
Note this is only looking at binary size, performance isn't considered / benched yet. It's also assumed that the overall memory footprint of a given binary isn't changed, IE going from -O3 to -Oz isn't going to prevent xz from eating 60MB of working ram to do it's job, just the binary's size in RAM will hopefully shrink. The theory here is on VERY RAM constrained systems, the loss of raw perf by going to a lower optimization level will be offset by not paging to swap as often by virtue of executable code being smaller.

I'm testing this by doing a full world rebuild in a chroot, just changing cflags to see what happens at the macro level. No performance testing has been done to compare the speed impact of these tweaks.

- Kernel 6.6.67 (Gzip)
  - Os bzImage 4440576bytes
  - O2 bzImage 5411328bytes
- World
  - /usr/lib
    - Oz 282596Kbytes
    - Os 384272Kbytes
    - O2 403928Kbytes
  - /usr/bin
    - Oz 72540Kbytes
    - Os 73088Kbytes
    - O2 108320Kbytes
  - /usr/libexec
    - Oz 182028Kbytes
    - Os 182044Kbytes
    - O2 182564Kbytes
