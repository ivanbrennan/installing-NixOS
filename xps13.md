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

## Boot from USB
- select default NixOS installer
- logs into root

Note: If you make a mistake during the subsequent steps and need to restart the installer, you can run `shutdown now` to cleanly shut the machine off. Don't use `reboot`, as that seems to get stuck.

## Partitions: before
Original partitioning scheme of the NVMe SSD as reported by `fdisk -l`.
```
Disk /dev/nvme0n1: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: DACEF394-88D5-415B-8C97-1CDBB7516C0A

Device             Start       End   Sectors   Size Type
/dev/nvme0n1p1      2048   1026047   1024000   500M EFI System
/dev/nvme0n1p2   1026048   1288191    262144   128M Microsoft reserved
/dev/nvme0n1p3   1288192 473675775 472387584 225.3G Microsoft basic data
/dev/nvme0n1p4 473675776 474597375    921600   450M Windows recovery environment
/dev/nvme0n1p5 474597376 497907711  23310336  11.1G Windows recovery environment
/dev/nvme0n1p6 497909760 500117503   2207744   1.1G Windows recovery environment
```

As reported by 'gdisk /dev/nvme0n1`
```
The protective MBR's 0xEE partition is oversized! Auto-repairing

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Disk /dev/nvme0n1: 500118192 sectors, 238.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): DACEF394-88D5-415B-8C97-1CDBB7516C0A
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 500118158
Partitions will be aligned on 2048-sector boundaries
Total free space is 4717 sectors (2.3 MiB)

Number  Start (sector)    End (sector)  Size      Code  Name
   1            2048         1026047   500.0 MiB  EF00  EFI system partition
   2         1026048         1288191   128.0 MiB  0C01  Microsoft reserved ...
   3         1288192       473675775   225.3 GiB  0700  Basic data partition
   4       473675776       474597375   450.0 MiB  2700
   5       474597376       497907711   11.1 GiB   2700
   6       497909760       500117503   1.1 GiB    2700
```

## Partitions: after
Our goal looks like this:
```
Number  Size    Code  Name
1       500 MB  EF00  EFI System Partition
2       rest    8E00  Linux LVM
```

Delete existing partitions
```
o
p
```

Add EFI System partition
```
n
1
<Enter>
+500M
ef00
```

Add Linux LVM partition
```
n
2
<Enter>
<Enter>
8e00
```

Double-check
```
p

Disk /dev/nvme0n1: 500118192 sectors, 238.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 33DEB725-2DF3-4673-A182-B5FCB48D92FA
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 500118158
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size      Code  Name
   1            2048         1026047   500.0 MiB  EF00  EFI System
   2         1026048       500117503   238.0 GiB  8E00  Linux LVM
```
Write changes
```
w
y
```
