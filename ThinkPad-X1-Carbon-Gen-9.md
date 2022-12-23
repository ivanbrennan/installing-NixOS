## Preparation

### Lenovo docs
https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-9th-gen-type-20xw-20xx/documentation/doc_userguide
https://psref.lenovo.com/
https://psref.lenovo.com/Product/ThinkPad/ThinkPad_X1_Carbon_Gen_9

#### UEFI BIOS updates

> Download and install the latest UEFI BIOS update package by one of the following methods:
> * Go to https://pcsupport.lenovo.com and select the entry for your computer.
>   Then, follow the on-screen instructions to download and install the latest UEFI
>   BIOS update package

#### Fedora
https://support.lenovo.com/us/en/solutions/pd031426
https://dl.fedoraproject.org/pub/alt/official-respins/33/

> Lenovo uses Fedora Workstation edition as our preloaded image. The only
> modification is documents in the /opt/lenovo directory.

To reinstall Fedora:
1. Go to: https://getfedora.org/ and download the Fedora Workstation installer.
1. Put the installer on a USB stick (instructions are on the Fedora site as to how to do this).
1. Put the USB stick in the computer, power on, and press F12 to launch the boot menu.
1. Select the USB stick and then follow the instructions.
1. Documents are on the Lenovo support site.

### References

#### Lenovo with Linux
https://forums.lenovo.com/t5/Linux-Operating-Systems/ct-p/lx_en
https://forums.lenovo.com/t5/Fedora/Frequently-Asked-Questions/m-p/5023454

#### Arch Wiki
https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_9)

#### Recent NixOS Release Notes
https://nixos.org/blog/announcements.html#21.05
https://nixos.org/blog/announcements.html#21.11

### nixos-hardware
https://github.com/NixOS/nixos-hardware/tree/master/lenovo/thinkpad/x1

#### NixOS Configurations
https://github.com/LEXUGE/nixos

### Issues to Watch For
- keyboard/trackpad on wake: https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_9)#TrackPoint/Keyboard/Trackpad_does_not_work_after_booting_into_the_desktop_or_waking_up_from_suspend_randomly
- https://discourse.nixos.org/t/first-installation-stuck-hang-for-thinkpad-x1-carbon-gen-9/12985
- suspend (older generation): https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_6)#Suspend_issues
- tpm related: https://wiki.archlinux.org/title/Lenovo_ThinkPad_T495
  - workaround is to turn off the Security Chip in BIOS

### NixOS Bootable USB

https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb
https://nixos.org/download.html#download-nixos

`wpa_supplicant` has a [bug](https://github.com/NixOS/nixpkgs/issues/154616)
that prevented it from working, and the minimal ISO doesn't include
NetworkManager. The graphical ISO works, but it causes the initial installation
to include Gnome and a lot of other software I don't want/need, leading to a
much longer installation process. I ended up using the minimal ISO along with
the ethernet connection.

Download and verify the ISO:
```sh
tmpDir="$(mktemp -d -t iso.XXXXXX)"
release="https://channels.nixos.org/nixos-21.11"
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
sudo dd \
    if=/dev/zero \
    of="${usb}" \
    bs=4M \
    status=progress

# Write
sudo dd \
    if="${tmpDir}/${iso}" \
    of="${usb}" \
    bs=4M \
    status=progress
```

Check the result:
```
sudo fdisk -l /dev/sda
```
```
Disk /dev/sda: 14.78 GiB, 15871246336 bytes, 30998528 sectors
Disk model: USB 3.0 FD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9fb6382f

Device     Boot Start     End Sectors  Size Id Type
/dev/sda1  *        0 1589247 1589248  776M  0 Empty
/dev/sda2       14256   90031   75776   37M ef EFI (FAT-12/16/32)
```

Clean up
```
rm -ri "$tmpDir"
```

## Configure ThinkPad startup
- power on ThinkPad
- hit Enter at Lenovo screen
- F1 to configure BIOS
 - I didn't end up changing any settings, but confirmed the following:
   - `Security` > `Secure Boot` > `Secure Boot` : `Off`
   - `Startup` > `Boot` : "USB CD" is first in the Boot Priority Order
 - There's a Security Chip setting that I left at `On`, but suspect could be turned `Off` without adverse affect

Note current UEFI BIOS info:
```
UEFI BIOS Version: N32ET75W (1.51)
UEFI BIOS Date (Year-Month-Day): 2021-12-02
Embedded Controller Version N32HT51W (1.31)
ME Firmware Version: 15.0.35.1951
EFI Secure Boot: Off
```

Save and exit.
Allow machine to boot.
On first boot, it automatically installed Fedora.
I let it do so, then shut it down.

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

Plug into ethernet.

Verify that sshd is running:
```
systemctl status sshd
```

Show the local IP address:
```
ip --brief address show
```

## SSH from other machine

```
ssh nixos@$THINKPAD_IP_ADDRESS
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

Disk /dev/loop0: 708.43 MiB, 742842368 bytes, 1450864 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 14.78 GiB, 15871246336 bytes, 30998528 sectors
Disk model: USB 3.0 FD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9fb6382f

Device     Boot Start     End  Size Id Type
/dev/sda1  *        0 1603584  783M  0 Empty
/dev/sda2       14260   75776   37M ef EFI (FAT-12/16/32)


Disk /dev/nvme0n1: 476.94GiB, 512110190592 bytes, 1000215216 sectors
Disk model: WDC PC SN730 SDBQNTY-512G-1001
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 94F31840-904C-46D6-B44C-FBC95C1E6374

Device           Start        End   Sectors   Size Type
/dev/nvme0n1p1    2048    1804287   1802240   880M EFI System
/dev/nvme0n1p2 1804288    3901439   2097152     1G Linux filesystem
/dev/nvme0n1p3 3901440 1000215182 996313743 475.1G Linux filesystem
```

```
# gdisk /dev/nvme0n1

GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/nvme0n1: 1000215216 sectors, 476.9 GiB
Model: WDC PC SN730 SDBQNTY-512G-1001
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 94F31840-904C-46D6-B44C-FBC95C1E6374
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 1000215182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)  End (sector)  Size       Code  Name
   1            2048       1804287   880.0 MiB   EF00  "EFI"
   2         1804288       3901439   1024.0 MiB  8300
   3         3901440    1000215182   475.1 GiB   8300
```

## Partitioning

We will only encrypt the Linux LVM partition as the boot process will need to
be able to read the EFI System Partition before prompting us for the encryption
key.

https://nixos.org/manual/nixos/stable/index.html#sec-installation-partitioning-UEFI
https://gist.github.com/martijnvermaat/76f2e24d0239470dd71050358b4d5134
https://dzone.com/articles/nixos-native-flake-deployment-with-luks-and-lvm

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
Model: WDC PC SN730 SDBQNTY-512G-1001 (nvme)
Disk /dev/nvme0n1: 512GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End    Size   File system  Name     Flags
 1      1049kB  537MB  536MB  fat32        boot     boot, esp
 2      537MB   512GB  512GB               primary  lvm
```

Double-check
```
gdisk /dev/nvme0n1

GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/nvme0n1: 1000215216 sectors, 476.9 GiB
Model: WDC PC SN730 SDBQNTY-512G-1001
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 9FFC6383-0931-456B-ADEF-86FCA8269F6D
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 1000215182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2669 sectors (1.3 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1048575   511.0 MiB   EF00  boot
   2         1048576      1000214527   476.4 GiB   8E00  primary
```

Triple-check
```
fdisk /dev/nvme0n1 -l

Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: WDC PC SN730 SDBQNTY-512G-1001
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9FFC6383-0931-456B-ADEF-86FCA8269F6D

Device           Start        End   Sectors   Size Type
/dev/nvme0n1p1    2048    1048575   1046528   511M EFI System
/dev/nvme0n1p2 1048576 1000214527 999165952 476.4G Linux LVM
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
mkfs.ext4 -L root /dev/cryptedpool/root
mkswap -L swap /dev/cryptedpool/swap
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

  boot.initrd.availableKernelModules = [ "xhci_pci" "thunderbolt" "nvme" "usb_storage" "sd_mod" ];
  boot.initrd.kernelModules = [ "dm-snapshot" ];
  boot.kernelModules = [ ];
  boot.extraModulePackages = [ ];

  fileSystems."/" =
    { device = "/dev/disk/by-uuid/dc71a88c-0ed9-42be-a036-8dfef5655434";
      fsType = "ext4";
    };

  fileSystems."/boot" =
    { device = "/dev/disk/by-uuid/D776-6EB3";
      fsType = "vfat";
    };

  swapDevices =
    [ { device = "/dev/disk/by-uuid/d93fdfca-1c91-40ec-8b4e-ed596a423cf0"; }
    ];

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

We'll eventually want to comment out the ethernet interface reference
([issue](https://github.com/NixOS/nixpkgs/issues/107908)), but for now, we need
it. (This was added automatically by `nixos-generate-config`.)
```
  networking.interfaces.enp0s13f0u2.useDHCP = true;
```

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
mkpasswd -m sha-512 > ivanpasswd
```

Edit the users configuration
```
  users.users.ivan = {
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

Select default configuration at the initial screen, and log in as ivan.

### troubleshooting

First let's just look for error logs:
```
[ivan@nixos:~]$ journalctl --priority err
-- Journal begins at Tue 2022-01-11 15:58:55 EST, ends at Tue 2022-01-11 16:26:13 EST. --
Jan 11 15:58:55 nixos kernel: x86/cpu: VMX (outside TXT) disabled by BIOS
Jan 11 15:58:55 nixos dhcpcd[927]: no valid interfaces found
Jan 11 15:59:25 nixos dhcpcd[927]: timed out
Jan 11 16:00:25 nixos systemd[1]: Timed out waiting for device /sys/subsystem/net/devices/enp0s13f0u2.
```

- `VMX (outside TXT) disabled by BIOS`
  - Harmless. Related to virtualization settings in BIOS.
    * https://askubuntu.com/a/1300834/744770
- `hpet_acpi_add: no address or irqs in _CRS`
  - Something related to battery info and/or event timers.
    * https://bugzilla.kernel.org/show_bug.cgi?id=203183
    * https://www.phoronix.com/scan.php?page=news_item&px=Linux-5.15-rc5-x86
    * https://bugs.launchpad.net/ubuntu/+source/acpi/+bug/1909133
- `usb: port power management may be unreliable`
  - Possibly also related to [ACPI](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface)
    * https://unix.stackexchange.com/a/323710/47044
- `dhcpcd[927]: no valid interfaces found`
  - Does this relate to the presence of enp0s13f0u2 in configuration.nix?
    * https://github.com/NixOS/nixpkgs/issues/74471
    * https://github.com/NixOS/nixpkgs/issues/109839
- `sof-audio-pci ...`
  - audio related
    * https://github.com/thesofproject/linux/issues/3206
    * https://forum.manjaro.org/t/no-sound-on-dell-latitude-9520/79495
- `dhcpcd[1071]: wlp0s20f3: no IPv6 Routers available`
  - not sure whether this is an issue or not.
    * https://github.com/NixOS/nixpkgs/issues/27111
    * https://github.com/NixOS/nixpkgs/pull/27688
    * https://nixos.org/manual/nixos/stable/index.html#sec-ipv6
- `ACPI: [Firmware Bug]: BIOS _OSI(Linux) query ignored`
  - Ignore
    * https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1275985/comments/4
- `acpi PNP0C14:01: duplicate WMI GUID 05901221-D566-11D1-B2F0-00A0C9062910 (first instance was on PNP0C14:00)`
  - Ignore
    * https://bugzilla.kernel.org/show_bug.cgi?id=201885#c10
- `thinkpad_acpi: Disabling thinkpad-acpi brightness events by default...`
  - Possibly indicates that the brightness keys won't work, but I haven't gotten that far yet
    * `acpi_backlight=video` https://wiki.archlinux.org/title/Backlight#Kernel_command-line_options
    * https://discourse.nixos.org/t/screen-brightness-keys-i3-lenovo-p14s-gen2-amd/14633/5
- `iwlwifi 0000:00:14.3: api flags index 2 larger than supported by driver`
  - watch for wifi problems
    * https://wiki.gentoo.org/wiki/Iwlwifi#Wireless_not_working
    * https://forums.gentoo.org/viewtopic-p-8486892.html#8486892
    * "I was missing the CONFIG_PCI_MSI kernel option"
- `kernel: thermal thermal_zone7: failed to read out thermal zone (-61)`
  - inconclusive
    * https://bugzilla.kernel.org/show_bug.cgi?id=201761
- `kernel: Bluetooth: hci0: Failed to read codec capabilities (-56)`
  - appears to be non-critical, and possibly fixed in more recent kernel
    * https://lkml.org/lkml/2022/1/3/96
- `systemd-udevd[782]: wlan0: Process '/nix/store/pbfraw351mksnkp2ni9c4rkc9cpp89iv-bash-5.1-p12/bin/sh -c 'echo 2 > /proc/sys/net/ipv6/conf/wlan0/use_tempaddr'' failed with exit code 1.`
  - something relating to udev rules, ethernet network interfaces, ipv6
    * https://github.com/NixOS/nixpkgs/issues/86764

### continuing configuration

Set `users.users.ivan.openssh.authorizedKeys.keys`
https://nixos.org/manual/nixos/stable/index.html#sec-ssh

Set `networking.hostName`

Set `hardware.trackpoint.enable`?
Set `hardware.trackpoint.emulateWheel`?

Add to imports:
```
"${builtins.fetchGit { url = "https://github.com/NixOS/nixos-hardware.git"; ref = "master"; rev = "d7a12fcc071bff59bd0ead589c975d802952a064"; }}/lenovo/thinkpad/x1/9th-gen"
```

Comment out `networking.interfaces.enp0s13f0u2`
https://github.com/NixOS/nixpkgs/issues/107908
https://github.com/NixOS/nixpkgs/issues/107908#issuecomment-882549381
Maybe I should also comment out `networking.interfaces.enp0s13f0u2.useDHCP` since I'm using NetworkManager?

Switch to the nixos-unstable channel
```
# nix-channel --add https://nixos.org/channels/nixos-unstable nixos
# nixos-rebuild switch --upgrade
```

```
services.xserver.displayManager.lightdm.enable = true;
services.xserver.displayManager.autoLogin.enable = true;
services.xserver.displayManager.autoLogin.user = "ivan";
```

To make Qt 5 applications look similar to GTK ones, you can use the following configuration:
```
qt5.enable = true;
qt5.platformTheme = "gtk2";
qt5.style = "gtk2";
```

Custom XKB layouts: https://nixos.org/manual/nixos/stable/index.html#custom-xkb-layouts

Emacs: https://nixos.org/manual/nixos/stable/index.html#module-services-emacs

https://nixos.org/manual/nixos/stable/index.html#sect-nixos-systemd-general

from other machine:
```
scp -r ~/Development/resources/localdots ivan@192.168.0.106:/home/ivan/localdots
```

from ThinkPad:
```
mkdir -p ~/Development/resources
mv ~/localdots ~/Development/resources/localdots
ln -svnf $HOME/Development/resources/localdots/ssh ~/.ssh
ln -svnf $HOME/Development/resources/localdots/bashrc.local ~/.bashrc.local

sudo -i
cd /etc/nixos
mkdir /root/backup-nixos-configuration
mv /etc/nixos/{configuration.nix,hardware-configuration.nix} \
    /root/backup-nixos-configuration/
GIT_SSH_COMMAND='ssh -i /home/ivan/.ssh/id_rsa -o IdentitiesOnly=yes' \
    git clone git@github.com:ivanbrennan/nixbox.git /etc/nixos/
git config user.email "ivan.brennan@gmail.com"
git config user.name "ivanbrennan"
git checkout -b thinkpad-x1-carbon-gen-9
git mv configuration.nix main-configuration.nix
git commit
mv /root/backup-nixos-configuration/{configuration.nix,hardware-configuration.nix}  ./
git commit
GIT_SSH_COMMAND='ssh -i /home/ivan/.ssh/id_rsa -o IdentitiesOnly=yes' \
    git push -u origin thinkpad-x1-carbon-gen-9
exit

# as user "ivan"
cd ~/Development/resources
git clone https://github.com/ivanbrennan/nixdots.git "$DOTFILES"
```

Copy gnupg from other machine:
```
scp -r ~/.gnupg ivan@192.168.0.110:/home/ivan/.gnupg
```

SSH back onto thinkpad
```
rm .gnupg/gpg-agent.conf
stow --restow --verbose --dir="$DOTFILES/stow" --target="$HOME" gpg
```

```
[remote "origin"]
        url = git@github.com:ivanbrennan/nixbox.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        pushurl = git@github.com:ivanbrennan/nixbox.git
        pushurl = git@bitbucket.org:ivanbrennan/nixbox.git
[branch "master"]
        remote = origin
        merge = refs/heads/master
[remote "github"]
        url = git@github.com:ivanbrennan/nixbox.git
        fetch = +refs/heads/*:refs/remotes/github/*
[remote "bitbucket"]
        url = git@bitbucket.org:ivanbrennan/nixbox.git
        fetch = +refs/heads/*:refs/remotes/bitbucket/*
```

https://github.com/xmonad/xmonad-contrib/pull/613

### Notes for next time

Use the minimal image and a wired connection.

Make a bootable USB with the full image, not the minimal, since that would allow
using NetworkManager.

Plug into a wired network connection before running `nixos-generate-config`, so
that it will automatically declare the network interface.

Looks interesting: https://chrishayward.xyz/dotfiles/
