## Download installation CD image

- [NixOS downloads page](https://nixos.org/nixos/download.html)
 - [Minimal installation CD, 64-bit Intel/AMD](https://d3g5gsiof5omrk.cloudfront.net/nixos/17.09/nixos-17.09.2075.ac35504065/nixos-minimal-17.09.2075.ac35504065-x86_64-linux.iso)

## Format USB drive (on a mac)

- insert USB drive in mac
 - click "Ignore" if warned about an unreadable device
- open Disk Utility
 - select the USB external drive
 - click "Erase"
     - Name: NixOS
     - Format: ExFAT
     - Scheme: GUID Partition Map

## Make bootable USB drive

```
diskutil list
diskutil unmountDisk /dev/disk2
sudo dd bs=4m \
        if=/Users/ivan/Downloads/nixos-minimal-17.09.2075.ac35504065-x86_64-linux.iso \
        of=/dev/disk2
```
When macOS prompts about unreadable disk, select "Eject".

## BIOS
- power on Dell XPS 13
- tap F2 repeatedly at Dell logo to enter BIOS
- disable Secure Boot
  - Settings > Secure Boot > Secure Boot Enable : Disabled
- adjust Boot Sequence
  - Settings > General > Boot Sequence : move Windows Boot Manager to bottom of list
- disable Intel hardware RAID and use AHCI instead
  - Settings > System Configuration > SATA Operation : AHCI
- apply (restarts and boots from usb)
