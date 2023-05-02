## Preparation

### Lenovo docs

X1 Carbon 5th Gen - Kabylake (Type 20HR, 20HQ) Laptop (ThinkPad) - Type 20HQ

- https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-type-20hr-20hq/20hq/20hqs1bx00/pf12yu37/documentation
- https://psref.lenovo.com/
- https://psref.lenovo.com/syspool/Sys/PDF/ThinkPad/ThinkPad_X1_Carbon_5th_Gen/ThinkPad_X1_Carbon_5th_Gen_Spec.PDF
- https://forums.lenovo.com/t5/ThinkPad-X-Series-Laptops/bd-p/tp02_en

#### Arch Wiki

- https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_5)
- troubleshooting https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_5)#Troubleshooting

#### Recent NixOS Release Notes

- https://nixos.org/manual/nixos/stable/release-notes.html#sec-release-22.05
- https://nixos.org/manual/nixos/stable/release-notes.html#sec-release-22.11

### nixos-hardware

https://github.com/NixOS/nixos-hardware/tree/master/lenovo/thinkpad/x1

#### NixOS Configurations

https://github.com/LEXUGE/nixos

### Change Boot Priority Order

Restart, hit Enter, hit F1 to enter BIOS

```
Startup
  Boot
    Boot Priority Order:
      1. Windows Boot Manager
      2. USB CD
      3. USB FDD
      4. NVMe0 KXG5AZNV256G TOSHIBA
      5. PCI LAN IBA CL Slot 00FE v0109
      6. ATA HDD0
      7. USB HDD
    Excluded from boot priority order:
      - Other CSD
      - Other HDD

  Network Boot:                PCI LAN
  UEFI/Legacy Boot:            Legacy Only
    CSM Support:               Yes
  Boot Mode:                   Quick
  Option Key Display:          Enabled
  Boot device List F12 Option: Enabled
  Boot Order Lock:             Disabled
```

Press F10 to Save and Exit

### Stop computer from shutting down when lid closed

- Log into windows.
- Start > Settings > System > Power & sleep
- Additional power settings > Choose what closing the lid does
  - When I close the lid
    - On battery: Hibernate
    - Plugged in: Hibernate
- Save Changes

Didn't fix the issue.

### Disable "fast startup" windows feature

- Log into windows.
- Start > Settings > System > Power & sleep > Additional power settings
- Choose what the power buttons do > Change settings that are currently unavailable
  - When I close the lid
    - On battery: Hibernate
    - Plugged in: Hibernate
- Save Changes

https://www.freecodecamp.org/news/how-to-dual-boot-any-linux-distribution-with-windows/

### Ensure that Secure Boot is disabled

### Change UEFI/Legacy Boot setting?

Restart, hit Enter, hit F1 to enter BIOS

```
Startup
  UEFI/Legacy Boot: Both
    UEFI/Legacy Boot Priority: Legacy First
    CSM Support:               Yes
```

Not sure this will work with the current windows setup, so I changed it back

```
Startup
  UEFI/Legacy Boot: Legacy Only
    CSM Support:    Yes
```

### Disable hibernate in windows

- Press the Windows button on the keyboard to open Start menu or Start screen.
- Search for cmd. In the search results list, right-click Command Prompt, and then select Run as Administrator.
- When you are prompted by User Account Control, select Continue.
- At the command prompt, type `powercfg.exe /hibernate off`, and then press Enter.
- Type exit, and then press Enter to close the Command Prompt window.

https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/disable-and-re-enable-hibernation#how-to-make-hibernation-unavailable

### Change UEFI/Legacy Boot setting again

Restart, hit F1 to enter BIOS

```
Startup
  UEFI/Legacy Boot: Both
    UEFI/Legacy Boot Priority: Legacy First
    CSM Support:               Yes
```

https://isdanni.com/ubuntu-18-04/

### Partitioning

In windows bar, search for "disk management", and open Disk Management utility.

Note that disk is using MBR
- Check MBR or GPT partition style from Disk Management
- Right-click the disk (not the partition) and select the Properties option.
- Click the Volumes tab. Check the “Partition style” field

Shrink the "(C:)" partition using Disk Management in Windows:
- Right-click the "(C:)" partition and select "Shrink Volume" from the context menu.
- Enter the amount of space you want to shrink the partition by: 128000 MB

### Find the Windows 10 Product Key

- Press the Windows button on the keyboard to open Start menu or Start screen.
- Search for cmd. In the search results list, right-click Command Prompt, and then select Run as Administrator.
- When you are prompted by User Account Control, select Continue.
- At the command prompt, type `wmic path softwarelicensingservice get OA3xOriginalProductKey`, and then press Enter.
- Type exit, and then press Enter to close the Command Prompt window.

Given that I could (in theory) re-install Windows using the above product key,
I think I'll simply install NixOS the way I've done in the past (UEFI, GPT),
removing Windows altogether.

misc links:
- https://www.thinkwiki.org/wiki/BIOS_Upgrade
- https://wiki.gentoo.org/wiki/Lenovo_ThinkPad_X1_Carbon_6th_generation
- https://www.reddit.com/r/thinkpad/wiki/series/x1series/

### Change Boot Priority Order (again)

Restart, hit Enter, hit F1 to enter BIOS

```
Startup
  Boot
    Boot Priority Order:
      1. USB CD
      2. USB FDD
      3. NVMe0 KXG5AZNV256G TOSHIBA
      4. Windows Boot Manager
      5. PCI LAN IBA CL Slot 00FE v0109
      6. ATA HDD0
      7. USB HDD
    Excluded from boot priority order:
      - Other CSD
      - Other HDD
```

- Change UEFI/Legacy Boot to `Both`
- Change UEFI/Legacy Boot Priority to `UEFI First`

Shut down. Start up.

### NixOS Bootable USB

- https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb
- https://nixos.org/download.html#nixos-iso

Download and verify the ISO:
```sh
tmpDir="$(mktemp -d -t iso.XXXXXX)"
release="https://channels.nixos.org/nixos-22.11"
iso="latest-nixos-minimal-x86_64-linux.iso"

# Download
wget -P "$tmpDir" "${release}/${iso}" "${release}/${iso}.sha256"

# Verify
sha256sum "${tmpDir}/${iso}"
cat "${tmpDir}/${iso}.sha256"
```

Insert USB drive, ensure it's not mounted, and identify the device:
```
$ lsblk
NAME                   MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                      8:0    1  14.8G  0 disk
└─sda1                   8:1    1  14.8G  0 part
nvme0n1                259:0    0 238.5G  0 disk
├─nvme0n1p1            259:1    0   500M  0 part  /boot
└─nvme0n1p2            259:2    0   238G  0 part
  └─root               254:0    0   238G  0 crypt
    ├─cryptedpool-swap 254:1    0     4G  0 lvm   [SWAP]
    └─cryptedpool-root 254:2    0   234G  0 lvm   /
```
Above, the device is `/dev/sda`

Erase the disk, then write the ISO to disk:
```
usb=/dev/sda

# Erase (optional)
# https://wiki.archlinux.org/title/Securely_wipe_disk
sudo fdisk -l /dev/sda
# Disk /dev/sda: 119.53 GiB, 128345702400 bytes, 250675200 sectors
# Disk model: Flash Drive
# Units: sectors of 1 * 512 = 512 bytes
# Sector size (logical/physical): 512 bytes / 512 bytes
# I/O size (minimum/optimal): 512 bytes / 512 bytes
#
# NOTE: We'll run dd using bs=4M, which means 4 * (1024 * 1024).
# The entire USB is 128345702400 bytes, so we'll want to use a
# count of 128345702400 / (4 * 1024 * 1024) = 30600.
sudo dd \
    if=/dev/zero \
    of="${usb}" \
    bs=4M \
    count=30600 \
    status=progress

# Write
sudo dd \
    if="${tmpDir}/${iso}" \
    of="${usb}" \
    bs=4M \
    conv=fsync

# Check the result
sudo fdisk -l "${usb}"
```
```
Disk /dev/sda: 119.53 GiB, 128345702400 bytes, 250675200 sectors
Disk model: Flash Drive
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9fb6382f

Device     Boot Start     End Sectors  Size Id Type
/dev/sda1  *        0 1697791 1697792  829M  0 Empty
/dev/sda2       16128   22271    6144    3M ef EFI (FAT-12/16/32)
```

Clean up
```
rm -ri "$tmpDir"
```

## Boot ThinkPad from USB

- insert USB
- press power
- hit Enter at Lenovo screen
- F12
- select USB drive
- select default NixOS installer

Installation instructions:
https://nixos.org/manual/nixos/stable/#sec-installation

## Set nixos password

We'll be performing most of the installation remotely via SSH.
To allow an SSH login, we need to set a password for the nixos user.
```
$ passwd nixos
```

## Networking

https://nixos.org/manual/nixos/stable/index.html#sec-installation-manual-networking

Check that we're on at least wpa_supplicant 2.10
```
wpa_supplicant -v
# wpa_supplicant v2.10

sudo systemctl start wpa_supplicant
wpa_cli

> add_network
0
> set_network 0 ssid "myhomenetwork"
OK
> set_network 0 psk "mypassword"
OK
> set_network 0 key_mgmt WPA-PSK
OK
> enable_network 0
OK
...
<3>CTRL-EVENT-CONNECTED - Connection to...
> quit
```

Verify that sshd is running:
```
systemctl status sshd
```

Show the local IP address:
```
ip --brief address show

...
wlp4s0    UP    192.168.0.109/24
```

## SSH from other machine

```
ssh nixos@192.168.0.109
```

## Gain root privileges

```
$ sudo -i
```

## Setup hard drive
https://nixos.org/manual/nixos/stable/#sec-installation-partitioning-UEFI

### Partitions: before

```
# fdisk -l

Disk /dev/loop0: 795.88 MiB, 834539520 bytes, 1629960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/nvme0n1: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: KXG5AZNV256G TOSHIBA
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0794dd56

Device         Boot     Start       End   Sectors   Size Id Type
/dev/nvme0n1p1 *         2048    104447    102400    50M  7 HPFS/NTFS/exFAT
/dev/nvme0n1p2         104448 236883022 236778575 112.9G  7 HPFS/NTFS/exFAT
/dev/nvme0n1p3      499027968 500113407   1085440   530M 27 Hidden NTFS WinRE


Disk /dev/sda: 119.53 GiB, 128345702400 bytes, 250675200 sectors
Disk model: Flash Drive
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9fb6382f

Device     Boot Start     End Sectors  Size Id Type
/dev/sda1  *        0 1697791 1697792  829M  0 Empty
/dev/sda2       16128   22271    6144    3M ef EFI (FAT-12/16/32)
```

```
# gdisk /dev/nvme0n1

GPT fdisk (gdisk) version 1.0.9

Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: not present


***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************


Command (? for help): p
Disk /dev/nvme0n1: 500118192 sectors, 238.5 GiB
Model: KXG5AZNV256G TOSHIBA
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): C0761BCC-A5B7-4884-BB6A-DF2C246284EB
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 500118158
Partitions will be aligned on 2048-sector boundaries
Total free space is 262151710 sectors (125.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          104447   50.0 MiB    0700  Microsoft basic data
   2          104448       236883022   112.9 GiB   0700  Microsoft basic data
   3       499027968       500113407   530.0 MiB   2700  Windows RE
```

## Partitioning

We will only encrypt the Linux LVM partition as the boot process will need to
be able to read the EFI System Partition before prompting us for the encryption
key.

- https://nixos.org/manual/nixos/stable/index.html#sec-installation-partitioning-UEFI
- https://gist.github.com/martijnvermaat/76f2e24d0239470dd71050358b4d5134
- https://dzone.com/articles/nixos-native-flake-deployment-with-luks-and-lvm

Create partitions:
```
parted --script --align optimal  \
    /dev/nvme0n1 --              \
    mklabel gpt                  \
    mkpart ESP fat32 1MiB 512MiB \
    set 1 boot on                \
    name 1 boot                  \
    mkpart primary 512MiB 100%   \
    set 2 lvm on                 \
    name 2 primary
```

Check result:
```
parted /dev/nvme0n1 print
```
```
Model: KXG5AZNV256G TOSHIBA (nvme)
Disk /dev/nvme0n1: 256GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End    Size   File system  Name     Flags
 1      1049kB  537MB  536MB  ntfs         boot     boot, esp
 2      537MB   256GB  256GB               primary  lvm
```

Double-check
```
gdisk /dev/nvme0n1

GPT fdisk (gdisk) version 1.0.9

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/nvme0n1: 500118192 sectors, 238.5 GiB
Model: KXG5AZNV256G TOSHIBA
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 98690081-C8C5-4D32-98DB-CD4BA9318053
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 500118158
Partitions will be aligned on 2048-sector boundaries
Total free space is 2669 sectors (1.3 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1048575   511.0 MiB   EF00  boot
   2         1048576       500117503   238.0 GiB   8E00  primary
```

Triple-check
```
fdisk /dev/nvme0n1 -l

Disk /dev/nvme0n1: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: KXG5AZNV256G TOSHIBA
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 98690081-C8C5-4D32-98DB-CD4BA9318053

Device           Start       End   Sectors  Size Type
/dev/nvme0n1p1    2048   1048575   1046528  511M EFI System
/dev/nvme0n1p2 1048576 500117503 499068928  238G Linux LVM
```

### Encryption

```
cryptsetup luksFormat /dev/disk/by-partlabel/primary
cryptsetup luksOpen /dev/disk/by-partlabel/primary crypted
```

### Logical volumes

```
pvcreate /dev/mapper/crypted
vgcreate cryptedpool /dev/mapper/crypted
lvcreate --name swap --size 6GB cryptedpool
lvcreate --name root --extents '100%FREE' cryptedpool
```

### Format

```
mkfs.fat -F 32 -n boot /dev/disk/by-partlabel/boot
# mkfs.fat 4.2 (2021-01-31)
# mkfs.fat: Warning: lowercase labels might not work properly on some systems

mkfs.ext4 -L root /dev/cryptedpool/root
# mke2fs 1.46.5 (30-Dec-2021)
# Creating filesystem with 60806144 4k blocks and 15204352 inodes
# Filesystem UUID: 6edfd726-c649-4127-9fb1-0b4c3f0a8ae7
# Superblock backups stored on blocks:
# 	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
# 	4096000, 7962624, 11239424, 20480000, 23887872
#
# Allocating group tables: done
# Writing inode tables: done
# Creating journal (262144 blocks): done
# Writing superblocks and filesystem accounting information: done

mkswap -L swap /dev/cryptedpool/swap
# Setting up swapspace version 1, size = 6 GiB (6442446848 bytes)
# LABEL=swap, UUID=f3598f8e-d99d-426b-b26e-8be7979371f1
```

### Mount

```
mount /dev/disk/by-label/root /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/boot /mnt/boot
swapon /dev/cryptedpool/swap
```

## Generate NixOS configuration

### Generate

```
nixos-generate-config --root /mnt
```

### Verify hardware-configuration

```
cat /mnt/etc/nixos/hardware-configuration.nix
```

```
# Do not modify this file!  It was generated by ‘nixos-generate-config’
# and may be overwritten by future invocations.  Please make changes
# to /etc/nixos/configuration.nix instead.
{ config, lib, pkgs, modulesPath, ... }:

{
  imports =
    [ (modulesPath + "/installer/scan/not-detected.nix")
    ];

  boot.initrd.availableKernelModules = [ "xhci_pci" "nvme" "usb_storage" "sd_mod" "rtsx_pci_sdmmc" ];
  boot.initrd.kernelModules = [ "dm-snapshot" ];
  boot.kernelModules = [ ];
  boot.extraModulePackages = [ ];

  fileSystems."/" =
    { device = "/dev/disk/by-uuid/6edfd726-c649-4127-9fb1-0b4c3f0a8ae7";
      fsType = "ext4";
    };

  fileSystems."/boot" =
    { device = "/dev/disk/by-uuid/2236-6C84";
      fsType = "vfat";
    };

  swapDevices =
    [ { device = "/dev/disk/by-uuid/f3598f8e-d99d-426b-b26e-8be7979371f1"; }
    ];

  # Enables DHCP on each ethernet and wireless interface. In case of scripted networking
  # (the default) this is the recommended approach. When using systemd-networkd it's
  # still possible to use this option, but it's recommended to use it in conjunction
  # with explicit per-interface declarations with `networking.interfaces.<interface>.useDHCP`.
  networking.useDHCP = lib.mkDefault true;
  # networking.interfaces.enp0s31f6.useDHCP = lib.mkDefault true;
  # networking.interfaces.wlp4s0.useDHCP = lib.mkDefault true;

  nixpkgs.hostPlatform = lib.mkDefault "x86_64-linux";
  powerManagement.cpuFreqGovernor = lib.mkDefault "powersave";
  hardware.cpu.intel.updateMicrocode = lib.mkDefault config.hardware.enableRedistributableFirmware;
}
```

### Edit configuration.nix

```
vim /mnt/etc/nixos/configuration.nix
```

#### luks

Write the root partition's UUID to a file.
```
blkid /dev/nvme0n1p2 \
  | grep -oP '(?<= UUID=")[^"]+(?=")' \
  > /root/uuid
```

Add `boot.initrd.luks.devices` to configuration.
```
  boot.initrd.luks.devices = {
    root = {
      device = "/dev/disk/by-uuid/<UUID here>";
      preLVM = true;
    };
  };
```

Delete the UUID file.
```
rm /root/uuid
```

#### networking

Enable NetworkManager
```
  networking.networkmanager.enable = true;
```

Enable OpenSSH daemon
```
  services.openssh.enable = true;
```

#### Set timezone

```
  time.timeZone = "America/New_York";
```

#### git, vim, wget

```
  environment.systemPackages = with pkgs; [
    git
    vim
    wget
  ];
```

#### user

Generate a hashed password for ivan
```
mkpasswd -m sha-512 > timpasswd
```

Edit the users configuration
```
  users.users.tim = {
    isNormalUser = true;
    uid = 1000;
    createHome = true;
    home = "/home/tim";
    extraGroups = [
      "wheel"
      "networkmanager"
    ];
    hashedPassword = <contents of timpasswd>;
  };
  users.mutableUsers = false;
```

Remove the hashed password file
```
rm timpasswd
```

#### result

The resulting /mnt/etc/nixos/configuration.nix:
```
# Edit this configuration file to define what should be installed on
# your system.  Help is available in the configuration.nix(5) man page
# and in the NixOS manual (accessible by running ‘nixos-help’).

{ config, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Use the systemd-boot EFI boot loader.
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  boot.initrd.luks.devices = {
    root = {
      device = "/dev/disk/by-uuid/61a7b394-322f-4a1e-9b61-fff4cbf9a208";
      preLVM = true;
    };
  };

  # networking.hostName = "nixos"; # Define your hostname.
  # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.
  networking.networkmanager.enable = true;

  # Set your time zone.
  time.timeZone = "America/New_York";

  # The global useDHCP flag is deprecated, therefore explicitly set to false here.
  # Per-interface useDHCP will be mandatory in the future, so this generated config
  # replicates the default behaviour.
  networking.useDHCP = false;
  networking.interfaces.enp0s13f0u2.useDHCP = true;
  networking.interfaces.wlp0s20f3.useDHCP = true;

  # Configure network proxy if necessary
  # networking.proxy.default = "http://user:password@proxy:port/";
  # networking.proxy.noProxy = "127.0.0.1,localhost,internal.domain";

  # Select internationalisation properties.
  # i18n.defaultLocale = "en_US.UTF-8";
  # console = {
  #   font = "Lat2-Terminus16";
  #   keyMap = "us";
  # };

  # Enable the X11 windowing system.
  # services.xserver.enable = true;




  # Configure keymap in X11
  # services.xserver.layout = "us";
  # services.xserver.xkbOptions = "eurosign:e";

  # Enable CUPS to print documents.
  # services.printing.enable = true;

  # Enable sound.
  # sound.enable = true;
  # hardware.pulseaudio.enable = true;

  # Enable touchpad support (enabled default in most desktopManager).
  # services.xserver.libinput.enable = true;

  # Define a user account. Don't forget to set a password with ‘passwd’.
  users.users.ivan = {
    isNormalUser = true;
    uid = 1000;
    createHome = true;
    home = "/home/ivan";
    extraGroups = [
      "wheel"
      "networkmanager"
    ];
    hashedPassword = "...";
  };
  users.mutableUsers = false;

  # List packages installed in system profile. To search, run:
  # $ nix search wget
  environment.systemPackages = with pkgs; [
    git
    vim
    wget
  ];

  # Some programs need SUID wrappers, can be configured further or are
  # started in user sessions.
  # programs.mtr.enable = true;
  # programs.gnupg.agent = {
  #   enable = true;
  #   enableSSHSupport = true;
  # };

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  services.openssh.enable = true;

  # Open ports in the firewall.
  # networking.firewall.allowedTCPPorts = [ ... ];
  # networking.firewall.allowedUDPPorts = [ ... ];
  # Or disable the firewall altogether.
  # networking.firewall.enable = false;

  # This value determines the NixOS release from which the default
  # settings for stateful data, like file locations and database versions
  # on your system were taken. It‘s perfectly fine and recommended to leave
  # this value at the release version of the first install of this system.
  # Before changing this value read the documentation for this option
  # (e.g. man configuration.nix or on https://nixos.org/nixos/options.html).
  system.stateVersion = "21.11"; # Did you read the comment?

}
```

## Install

```
nixos-install
```

## Shut down

End the SSH session, then shut down the ThinkPad.
```
shutdown now
```

Remove USB after shutdown.

Power on.

Select default configuration at the initial screen, and log in as tim.

Set up wifi connection:
```sh
nmcli --ask device wifi connect <TAB>
```

## sshd

```sh
ip --brief address show
```

Use `scp` to copy public key from client to server
```sh
scp ~/.ssh/id_rsa.pub tim@<IP-ADDRESS>:id_rsa.pub
```

On server, edit /etc/nixos/configuration.nix
- add public key to `users.users.tim.openssh.authorizedKeys.keys`
- `services.openssh.passwordAuthentication = false`
- `services.openssh.permitRootLogin = "no"`
- `services.openssh.kbdInteractiveAuthentication = false`
- extraConfig (AllowAgentForwarding, StreamLocalBindUnlink)

- https://github.com/ivanbrennan/nixbox/commit/4ed6c96e6751d16e304b9b0c32acee1222bdb206
- https://github.com/ivanbrennan/nixbox/commit/eab9cea2707bb50a3090e030eb6d441bcb993b83
- https://github.com/ivanbrennan/nixbox/commit/0539af157e1d700de3cd029e5cd8065254ebe63f
- https://github.com/ivanbrennan/nixbox/commit/7cf7d16ef922f6a0f3748d6bd8863d818b693a03

rebuild
```sh
nixos-rebuild switch
```

note this issue about possible sshd failures:
https://github.com/NixOS/nixpkgs/issues/98842

test log in from client machine:
```sh
ssh tim@<IP-ADDRESS>
```

remove copied public key
```sh
rm /home/tim/id_rsa.pub
```

## Git, ssh

on physical machine, create an SSH key:
```sh
ssh-keygen -t ed25519 -C "ivan.brennan+oakwood@gmail.com"
```

copy public key from remote machine via ssh:
```sh
ssh tim@<IP-ADDRESS> cat .ssh/id_ed25519.pub | tr -d '\n' | xsel --clipboard
```
add ssh key to codeberg
verify key in codeberg

put configuration.nix under git control:
```sh
sudo -i
cd /etc/nixos
git init -b main
git config user.email "ivan.brennan@gmail.com"
git config user.name "ivanbrennan"
git config core.sshCommand 'ssh -i /home/tim/.ssh/id_ed25519 -o IdentitiesOnly=yes'
git commit --allow-empty -m zero
git add configuration.nix hardware-configuration.nix
git commit -m 'initial configuration'
git remote add origin git@codeberg.org:ivanbrennan/oakwood-nixos.git
git push -u origin main
```

I'll defer setting up ssh-agent or gpg-agent for now.
I'm not yet sure whether it makes sense to do so.

### troubleshooting

First let's just look for error logs:
```
[tim@nixos:~]$ journalctl --priority err
Apr 22 22:44:07 nixos kernel: x86/cpu: VMX (outside TXT) disabled by BIOS
Apr 22 22:44:07 nixos kernel: x86/cpu: SGX virtualization disabled due to lack of VMX.
Apr 22 22:44:09 nixos kernel: snd_hda_codec_conexant hdaudioC0D0: vmaster hook already present before cdev!
Apr 22 22:44:09 nixos kernel: ucsi_acpi USBC000:00: UCSI_GET_PDOS failed (-95)
Apr 22 22:44:10 nixos kernel: Bluetooth: hci0: Reading supported features failed (-16)
```

- `VMX (outside TXT) disabled by BIOS`
  - Harmless. Related to virtualization settings in BIOS.
    - https://askubuntu.com/a/1300834/744770
    - https://askubuntu.com/a/1458737/744770
- `SGX virtualization disabled due to lack of VMX`
  - Harmless. Related to virtualization settings in BIOS.
    - https://patchwork.kernel.org/project/intel-sgx/patch/b3329777076509b3b601550da288c8f3c406a865.1616136308.git.kai.huang@intel.com/
- `snd_hda_codec_conexant hdaudioC0D0: vmaster hook already present before cdev!`
  - something to do with audio/mic mute?
    - https://unix.stackexchange.com/questions/729092/error-vmaster-hook-already-present-before-cdev
- `ucsi_acpi USBC000:00: UCSI_GET_PDOS failed (-95)`
  - something to do with USB-C, possibly power?
    - https://lore.kernel.org/all/006e3315b5b18fc8b3d0df1e3fc5a0072316b278.camel@gmail.com/T/
- `Bluetooth: hci0: Reading supported features failed (-16)`
  - likely due to the fact that I haven't enabled bluetooth

#### UEFI BIOS updates

Adjust BIOS setting to allow downgrading to older BIOS (in case I end having to undo the upgrade)
- Security
  - UEFI BIOS Update Option
    - Secure RollBack Prevention: Disabled

The version that's currently installed is N1MET41W (1.26)
- https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-type-20hr-20hq/20hq/20hqs1bx00/pf12yu37/downloads/ds120390-bios-update-utility-bootable-cd-for-windows-10-81-64-bit-7-32-bit-64-bit-linux-thinkpad-x1-carbon-type-20hq-20hr-20k3-20k4
- https://download.lenovo.com/pccbbs/mobiles/n1mur11w.iso
- https://download.lenovo.com/pccbbs/mobiles/n1mur11w.txt

> Download and install the latest UEFI BIOS update package by one of the following methods:
> * Go to https://pcsupport.lenovo.com and select the entry for your computer.
>   Then, follow the on-screen instructions to download and install the latest UEFI
>   BIOS update package

- https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-type-20hr-20hq/20hq/20hqs1bx00/pf12yu37
- https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-type-20hr-20hq/20hq/20hqs1bx00/pf12yu37/downloads/driver-list/component?name=BIOS%2FUEFI
- https://download.lenovo.com/pccbbs/mobiles/n1mur42w.iso
- https://download.lenovo.com/pccbbs/mobiles/n1mul42w.txt
- https://www.cyberciti.biz/faq/update-lenovo-bios-from-linux-usb-stick-pen/
- https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_5)#USB
- https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_6)#Manual_(El_Torito)
- install nixos
- update bios

```sh
# NOTE: do we also need the genisoimage util (provided by cdrkit)?
cd firmware
nix-shell -p geteltorito
geteltorito -o n1mur42w.img n1mur42w.iso
```

Insert USB drive, verify it's device name (in this case `/dev/sda`)
```sh
sudo fdisk -l /dev/sda
#
# Disk /dev/sda: 14.78 GiB, 15871246336 bytes, 30998528 sectors
# Disk model: USB 3.0 FD
# Units: sectors of 1 * 512 = 512 bytes
# Sector size (logical/physical): 512 bytes / 512 bytes
# I/O size (minimum/optimal): 512 bytes / 512 bytes
# ...
#
# NOTE: We'll run dd using bs=4M, which means 4 * (1024 * 1024).
# The entire USB is 15871246336 bytes, so we'll want to use a
# count of 15871246336 / (4 * 1024 * 1024) = 3784.
sudo dd \
    if=/dev/zero \
    of=/dev/sda \
    bs=4M \
    count=3784 \
    status=progress

sudo dd if=n1mur42w.img of=/dev/sda bs=512K
```
Remove USB drive

Shut down the gen 5 machine
```sh
shutdown now
```

- Insert USB into gen 5 machine.
- Power on, interrupt boot process with F12 to enter Boot Menu.
- Select USB
- Follow on-screen instructions carefully.
- Note: The machine automatically reboots a couple times during this process. Let it do its thing, don't interrupt it.
- It will eventually reboot into linux as per normal. Log in to prove that you still can, then finally shut down.
- Remove USB and power on.

- Reboot again and interrupt with F1 to go to BIOS settings.
- Re-enable Secure RollBack Prevention:
```
- Security
  - UEFI BIOS Update Option
    - Secure RollBack Prevention: Enabled
```

#### Additional configuration

set hostname

desktop environment
```nix
services.xserver.enable = true;

services.xserver.desktopManager.xfce.enable = true;
# services.xserver.desktopManager.plasma5.enable = true;
# services.xserver.desktopManager.gnome.enable = true;
```

software
- browser
- email

gtk themes
- Os Catalina Gtk https://www.xfce-look.org/p/1316421
- Graphite gtk theme https://www.xfce-look.org/p/1598493

https://github.com/davatorium/rofi

#### Additional things

Fine-tuning power consumption: tl;dr unclear whether this is still necessary or desirable.
- https://github.com/CRAG666/dotfiles/tree/main/thinkpad
- https://discourse.nixos.org/t/cpu-frequency-governor-doesn-t-seem-to-work/19315/7
- https://community.intel.com/t5/Processors/what-s-the-difference-between-p-state-and-speedstep/m-p/666688#M39091
- https://wiki.archlinux.org/title/CPU_frequency_scaling
- https://github.com/AdnanHodzic/auto-cpufreq/issues/335
- https://www.reddit.com/r/NixOS/comments/10adqyb/what_do_people_use_to_manage_their_cpu_frequency/
- https://www.reddit.com/r/NixOS/comments/10adqyb/comment/j44ast9/?utm_source=reddit&utm_medium=web2x&context=3
- what about setting the cpu frequency governor to ondemand?

Disabling Intel AMT:
- https://github.com/mjg59/mei-amt-check/
- https://superuser.com/questions/1195561/how-to-completely-disable-intel-amt-intel-me
- https://mattermedia.com/blog/disabling-intel-amt/
- I disabled AMT in BIOS. Things appear stable so far, but I'll give it a couple days to be safe before I perminantly disable it.

drivers
https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-type-20hr-20hq/20hq/20hqs1bx00/pf12yu37/downloads/driver-list
- The only one is an update of the ME (Management Engine) firmware, but I'm going to skip that because:
  - I'm going to permanently disable AMT
  - The ME update README mentions that it won't be able to run the update if the drive is encrypted

declarative gnome configuration: https://determinate.systems/posts/declarative-gnome-configuration-with-nixos

adjusting touchpad scroll speed:
- https://discourse.nixos.org/t/best-place-to-configure-xinput-commands/5349
- https://askubuntu.com/questions/1413750/how-to-change-2-finger-touchpad-scroll-speed-on-ubuntu-22-04
- https://forum.manjaro.org/t/adjust-mouse-scroll-speed/36634/4

SysRq key
- https://wiki.archlinux.org/title/keyboard_shortcuts
- https://en.wikipedia.org/wiki/Magic_SysRq_key
- https://superuser.com/a/1237766/162069
- https://superuser.com/questions/562348/altsysrq-on-a-laptop/1237766#comment2643227_1237766

#### Wake on Wireless LAN

I couldn't get this to work.

https://www.cyberciti.biz/faq/configure-wireless-wake-on-lan-for-linux-wifi-wowlan-card/

host
```sh
iw list | grep WoW -A10
iw phy0 wowlan show
sudo iw phy0 wowlan enable magic-packet
```

client
```sh
wakeonlan <mac-address>
sudo arp-scan --localnet --quiet | grep <mac-address>
ssh tim@192.168.0.101
```

- https://unix.stackexchange.com/questions/617848/how-to-wake-on-ssh-with-networkmanager
- https://discourse.ubuntu.com/t/wake-on-wlan/19938
- https://developer-old.gnome.org/NetworkManager/stable/NetworkManager.conf.html
