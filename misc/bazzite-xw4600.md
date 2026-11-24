Putting Bazzite where it doesn't want to go, on an HP XW4600.

Ye olde Core 2 Quad Q9550, 4GB of DDR2 ECC RAM and an NVIDIA GTX460, a true gaming powerhouse, a long time ago. It's x86_64 ABI v1, it's only got half the ram Bazzite wants as a minimum, and it's lacking UEFI support of any kind.

RAM: This box is PICKY, it wants DDR2 667 or 800, ECC, UNBUFFERED, x8 layout. It can support 16GB maximum if you can find it, at the end of 2026 it's not cheap, best I can find so far is $85/ea per 4GB stick so... for now skipping a direct RAM upgrade. Instead, I'm pressing into service a comperably old Gigabyte i-RAM SSD with 4GB of DDR to act as the swap. Slow, bottle necked by only supporting SATA 1.5Gb/s speed, it has no write endurance issues though, consistent latency, so I'll let it stand in. It's battery has long since left the chat so it's only good for literal volatile use like swap, will script wiping and recreating it at boot. An ACard Tech ANS-9010 would be soooo awesome for this job...

UEFI: Clover burned to an actual CDR, in a DVD-RW drive hanging off USB2 gets me the ability to boot UEFI, might even be able to fake a TPM for 'secure' boot with it? Fires up the Bazzite installer just fine, no complaints about spotting CSM/Legacy boot in use, trick will be getting Clover installed on the local HD after, that's where I've hit the wall before.
