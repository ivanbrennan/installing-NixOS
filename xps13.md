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

### Take note of which partition is which

Run `fdisk -l`
EFI System partition is `/dev/nvme0n1p1`
Linux LVM partition is `/dev/nvme0n1p2`

### Encryption

```
cryptsetup luksFormat /dev/nvme0n1p2
cryptsetup luksOpen /dev/nvme0n1p2 crypted
```

### Logical volumes

```
pvcreate /dev/mapper/crypted
vgcreate cryptedpool /dev/mapper/crypted
lvcreate --name swap --size 4GB cryptedpool
lvcreate --name root --extents '100%FREE' cryptedpool
```

### Format

```
mkfs.vfat -n BOOT /dev/nvme0n1p1
mkfs.ext4 -L root /dev/cryptedpool/root
mkswap -L swap /dev/cryptedpool/swap
```

### Mount

```
mount /dev/cryptedpool/root /mnt
mkdir /mnt/boot
mount /dev/disk/by-label/BOOT /mnt/boot
swapon /dev/cryptedpool/swap
```

## Generate NixOS configuration

### Generate

```
nixos-generate-config --root /mnt
```

### Verify hardware-configuration

```
$ cat /mnt/etc/nixos/hardware-configuration.nix
# Do not modify this file!  It was generated by ‘nixos-generate-config’
# and may be overwritten by future invocations.  Please make changes
# to /etc/nixos/configuration.nix instead.
{ config, lib, pkgs, ... }:

{
  imports =
    [ <nixpkgs/nixos/modules/installer/scan/not-detected.nix>
    ];

  boot.initrd.availableKernelModules = [ "xhci_pci" "nvme" "usb_storage" "sd_mod" "rtsx_pci_sdmmc" ];
  boot.kernelModules = [ "kvm-intel" ];
  boot.extraModulePackages = [ ];

  fileSystems."/" =
    { device = "/dev/disk/by-uuid/ff3c5531-3852-4210-9a65-f2b0055afc1c";
      fsType = "ext4";
    };

  fileSystems."/boot" =
    { device = "/dev/disk/by-uuid/04D7-1796";
      fsType = "vfat";
    };

  swapDevices =
    [ { device = "/dev/disk/by-uuid/61e9f2ad-a468-472f-9b1c-cc0dcf2feadb"; }
    ];

  nix.maxJobs = lib.mkDefault 8;
  powerManagement.cpuFreqGovernor = "powersave";
}
```

### Append LVM partition UUID to configuration.nix

```
blkid /dev/nvme0n1p2 \
  | grep -oP '(?<= UUID=")[^"]+(?=")' \
  >> /mnt/etc/nixos/configuration.nix
```

### Edit configuration.nix

`nano /mnt/etc/nixos/configuration.nix`

Add this
```
boot.initrd.luks.devices = [
  {
    name = "root";
    device = "/dev/disk/by-uuid/<the aforementioned UUID here>";
    preLVM = true;
  }
];
```
and set the timezone
```
time.timeZone = "America/New_York";
```

Should look like this:
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

  boot.initrd.luks.devices = [
    {
      name = "root";
      device = "/dev/disk/by-uuid/338a237d-0db7-4520-8ff5-3188533c59f6";
      preLVM = true;
    }
  ];

  # networking.hostName = "nixos"; # Define your hostname.
  # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.

  # Select internationalisation properties.
  # i18n = {
  #   consoleFont = "Lat2-Terminus16";
  #   consoleKeyMap = "us";
  #   defaultLocale = "en_US.UTF-8";
  # };

  # Set your time zone.
  time.timeZone = "America/New_York";

  # List packages installed in system profile. To search by name, run:
  # $ nix-env -qaP | grep wget
  # environment.systemPackages = with pkgs; [
  #   wget vim
  # ];

  # Some programs need SUID wrappers, can be configured further or are
  # started in user sessions.
  # programs.bash.enableCompletion = true;
  # programs.mtr.enable = true;
  # programs.gnupg.agent = { enable = true; enableSSHSupport = true; };

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  # services.openssh.enable = true;

  # Open ports in the firewall.
  # networking.firewall.allowedTCPPorts = [ ... ];
  # networking.firewall.allowedUDPPorts = [ ... ];
  # Or disable the firewall altogether.
  # networking.firewall.enable = false;

  # Enable CUPS to print documents.
  # services.printing.enable = true;

  # Enable the X11 windowing system.
  # services.xserver.enable = true;
  # services.xserver.layout = "us";
  # services.xserver.xkbOptions = "eurosign:e";

  # Enable touchpad support.
  # services.xserver.libinput.enable = true;

  # Enable the KDE Desktop Environment.
  # services.xserver.displayManager.sddm.enable = true;
  # services.xserver.desktopManager.plasma5.enable = true;

  # Define a user account. Don't forget to set a password with ‘passwd’.
  # users.extraUsers.guest = {
  #   isNormalUser = true;
  #   uid = 1000;
  # };

  # This value determines the NixOS release with which your system is to be
  # compatible, in order to avoid breaking some software such as database
  # servers. You should change this only after NixOS release notes say you
  # should.
  system.stateVersion = "17.09"; # Did you read the comment?

}
```

## Install

```
nixos-install
```

## Reboot

```
reboot
```

Select default configuration at the initial screen, and log in as root.

## Start configuring, setting up desktop, user accounts, etc...

First, did we get this far? Can we reliably (re)boot and log in as root?

Add vim so we can edit configuration.nix more easily, and mkpasswd so we can create a non-root user with a hashed password.
```
nano /etc/nixos/configuration.nix
```
```
environment.systemPackages = with pkgs; [
  mkpasswd
  vimHugeX
];
```

Rebuild NixOS
```
nixos-rebuild switch
```

Generate a hashed password for ivan
```
mkpasswd -m sha-512 > ivanpasswd
```

Edit the users configuration
```
vim /etc/nixos/configuration.nix ivanpasswd
```
```
users.extraUsers.ivan = {
  isNormalUser = true;
  uid = 1000;
  createHome = true;
  home = "/home/ivan";
  extraGroups = [
    "wheel"
    "networkmanager"
  ];
  hashedPassword = <contents of ivanpasswd>;
};
users.mutableUsers = false;
```

Remove the hashed password file
```
rm ivanpasswd
```

Rebuild NixOS
```
nixos-rebuild switch
```

Exit root account and login as ivan.

## Set up desktop manager

```
sudo vim /etc/nixos/configuration.nix
```
```
# Enable the X11 windowing system.
services.xserver.enable = true;
services.xserver.layout = "us";
...

# Enable touchpad support.
services.xserver.libinput.enable = true;

# Gnome desktop
services.xserver.desktopManager = {
  gnome3.enable = true;
  default = "gnome3";
};
```
Rebuild NixOS
```
sudo nixos-rebuild switch
```
Reboot
```
reboot
```

Log into Gnome as ivan. Connect to wifi.

## Git

Add `git` to `environment.systemPackages`.
Rebuild NixOS.
Put `/etc/nixos/` under version control
```
sudo -i
cd /etc/nixos
git init
echo '/hardware-configuration.nix' > .gitignore
git add configuration.nix .gitignore
git config user.email "ivan.brennan@gmail.com"
git config user.name "ivanbrennan"
git config core.editor "vim"
git commit -m 'initial commit'
exit
```
