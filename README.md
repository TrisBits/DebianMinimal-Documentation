# Debian 13 Trixie - Minimal Install with btrfs and zram

This document follows the steps, as presented in a video by **JustAGuy Linux**.  All credit and thanks goes to him.

Video Reference: [Debian 13 Trixie Minimal Install w/BTRFS (JustAGuy Linux)](https://www.youtube.com/watch?v=_zC4S7TA1GI)

It installs a minimal installtion of Debian with the btrfs file system.  There will be no graphical user interface, though one can be installed after if desired.

## Installation Steps

- Use the current Debian NetInstall (netinst) download. [https://www.debian.org/distrib/](https://www.debian.org/distrib/)
- Create bootable USB.
- Boot the system from the created USB to load the installer.

- Select "Advanced options".
- Select "Expert install".
- Work through the installation process as follows:
  - Choose Language > English
  - Select your location > Canada (Or whatever matches yours)
  - Configure locales > Canada  (Or whatever matches yours)
  - Additional locales > No Selections
- Choose the next step in the install process > Go down one item to Configure the keyboard
- Configure the keyboard > American English  (Or whatever matches yours)
- Detect and mount installation media > press Enter, when pop-up appears select Continue.
- Load installer components from installation media > press Enter, when pop-up appears leave all unchecked and select Continue.
- Detect network hardware > press Enter
- Configure the network > press Enter, when popup happens click Enter, then on next pop-up select the network interface you wish to use and Yes to Auto-configure networking.
    - enter a Hostname you wish then Continue.
    - Domain Name set to blank.
- Set up users and passwords
  - Allow login as root > No
  - Create user that will have sudo rights.  You will enter name, username, and password.
- Configure the clock
  - Set the clock using NTP > Yes
  - NTP server to use > time.nrc.ca  (Can also use default or one of your choosing)
  - Time Zone > UTC  (Suggest using UTC for servers or local time if will be used for a workstation)
- Detect disks > select Enter
- Partition disks > Manual
  - Select your disk that will be used for the OS.
  - Create new empty partition table on this device > Yes
  - Partition table type > gpt
  - Select the FREE SPACE (one down arrow from the disk previously selected) > Create a new partition
    - New partition size > 1g  (this will be for the EFI partition)
    - Location for the new partition > Beginning
    - Partition settings > Select Ext4 journaling line, change to **EFI System Partition**
    - Ensure bootable flag is ON
    - Select **Done setting up the partition**
    - Select the large FREE SPACE > Create New partition
    - New partition size > Enter to select the entire amount
    - Partition settings > Select Ext4 journaling line, change to **btrfs journaling file system**
    - Select **Done setting up the partition**
    - Keep remaining settings as their defaults
  - Finish partitioning and write changes to disk > select and Enter, on pop-up about swap space select No.  Then confirm Yes to write changes.

> ***NOTE:*** **DO NOT press enter for "Install the base system"**

- Press **Ctrl + Alt + F2** , this will enter into console mode.
- Execute the following commands in the console.
  - df -h (to list the disk partition names)
  - umount /target/boot/efi/
  - umount /target/
  - mount /dev/sda2 /mnt (the disk name may be different on your system, see results of your df -h command)
  - cd mnt/
  - ls
  - mv @rootfs @  (moving for possible use with timeshift)
  - ls
  - btrfs subvolume create @home
  - btrfs subvolume create @snapshots (you can use the up arrow to bring up the previous command to edit)
  - btrfs subvolume create @log
  - btrfs subvolume create @cache
  - ls  (Ctrl + L can be used to clear the screen if desired)
  - mount -o noatime,compress=zstd,subvol=@ /dev/sda2 /target
  - mkdir -p /target/boot/efi
  - mkdir -p /target/home
  - mkdir -p /target/.snapshots
  - mkdir -p /target/var/log
  - mkdir -p /target/var/cache
  - mount -o noatime,compress=zstd,subvol=@home /dev/sda2 /target/home
  - mount -o noatime,compress=zstd,subvol=@snapshots /dev/sda2 /target/.snapshots
  - mount -o noatime,compress=zstd,subvol=@log /dev/sda2 /target/var/log
  - mount -o noatime,compress=zstd,subvol=@cache /dev/sda2 /target/var/cache
  - mount /dev/sda1 /target/boot/efi
  - nano /target/etc/fstab
    - Go to line with UUID= ...  alter btrfs portion to **/ btrfs noatime,compress=zstd,subvol=@ 0 1**
    - Ctrl + K to cut the line, then Ctr + U five times
    - Update the pasted lines, starting with the second line as follows:
      - **/home btrfs noatime,compress=zstd,subvol=@home 0 2**
      - **/.snapshots btrfs noatime,compress=zstd,subvol=@snapshots 0 2**
      - **/var/log btrfs noatime,compress=zstd,subvol=@log 0 2**
      - **/var/cache btrfs noatime,compress=zstd,subvol=@cache 0 2**
    - Ctrl + O to write, Enter, Ctrl + X to exit
  - Ctrl + Alt + F1 , to exit out of console mode
- Install the base system > press Enter
- Kernel to install > press Enter on the default selection (linux-image-amd64)
- Drivers to include in the initrd > generic
- Configure the package manager
  - Use a network mirror > Yes
  - Protocol for file downloads > http
  - Debian archive mirror country > United States
  - Debian archive mirror > deb.debian.org
  - HTTP proxy information > leave blank
  - Use non-free firmware > Yes
  - Use non-free softwaere > Yes
  - Enable source repositories in APT > No
  - Services to use > select all including **backported software**
- Select and install software
  - Updates management on this system > No automatic updates (or your personal preference)
  - Participate in the package usage survey > No
  - Choose software to install > Remove all selections other than **SSH Server** (if installing for a server) and **Standard system utilities** selected
- Install the GRUB boot loader > Enter
  - Force GRUB installation to the EFI removable media path > Yes
  - Update NVRAM variables to automatically boot into Debian > Yes
  - Run os-prober automatically to detect and boot other OSes > No (if a dedicated system, no dual boot)
- Finish the installation > Enter
  - Is the system clock set to UTC > Yes
  - Reboot to Continue > Enter (remove usb)

## Post Reboot - Updates and ZRAM Configuration

- Login
- Execute the following commands.
  - lsblk (lists all your disks and sub volumes)
  - sudo apt update
  - sudo apt upgrade (if anything)
  - sudo apt install zram-tools
  - sudo nano /etc/default/zramswap
    - Confirm **ALGRO=lz4** is uncommented
    - Convirm **PERCENT** is uncommented and adjust as required, **PERCENT=25** (Depends on amount of RAM, ideally want around 8-10GB)  OR alter to a specific amount **SIZE=8192**
    - Ctrl + O to write, Enter, Ctrl + X to exit
  - sudo reboot
  - lsblk (to verify amount of zram post reboot)
  
