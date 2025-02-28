# Debian 12 Bookworm - Minimal Install with btrfs and zram

This document follows the steps, as presented in a video by **JustAGuy Linux**.  All credit and thanks goes to him.

Video Reference: [Debian 12 Bookworm Minimal Install w/BTRFS (JustAGuy Linux)](https://www.youtube.com/watch?v=MoWApyUb5w8)

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
- Access software for a blind person using a braille display > press Enter to continue
- Configure the keyboard > American English  (Or whatever matches yours)
- Detect and mount installation media > press Enter, when popup appears select Continue.
- Load installer components from installation media > press Enter, when popup appears leave all unchecked and select Continue.
- Detect network hardware > press Enter, when popup happens click Enter, then on next popup select the network interface you wish to use and Yes to Auto-configure networking.
  - Configure the network
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
  - Select the FREE SPACE > Create a new partition
    - New partition size > 300M
    - Location for the new partition > Beginning
    - Partition settings > Select Ext4 journaling line, change to EFI System Partition
    - Ensure bootable flag is ON
    - Select the large FREE SPACE > Create New partition
    - New partition size > Enter to select the entire amount
    - Partition settings > Select Ext4 journaling line, change to btrfs journaling file system
    - Keep remaining settings as their defaults
  - Finish partitioning and write changes to disk > select and Enter, on popup about swap space select No.  Then confirm Yes to write changes.

> ***NOTE:*** **DO NOT press enter for "Install the base system"**

- Press Ctrl + Alt + F2 , this will enter into console mode.
- Execute the following commands in the console.
  - df -h (to list the disk partition names)
  - umount /target/boot/efi/
  - umount /target/
  - mount /dev/sda2 /mnt/
  - cd mnt/
  - ls
  - mv @rootfs @  (moving for use with timeshift)
  - btrfs subvolume create @home
  - ls
  - mount -o rw,noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/sda2 /target/
  - mkdir -p /target/boot/efi/
  - mkdir -p /target/home
  - mount -o rw,noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@home /dev/sda2 /target/home
  - mount /dev/sda1 /target/boot/efi/
  - nano /target/etc/fstab
    - Go to line with UUID= ...  alter btrfs portion to **/ btrfs   rw,noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ 0 0**
    - Ctrl + K to cut the line, then Ctr + U twice
    - In the second pasted line, alter it to match **/home btrfs   rw,noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@home 0 0**
    - Ctrl + O to write, Ctrl + X to exit
  - Ctrl + Alt + F1 , to exit out of console mode
- Install the base system > press Enter
- Kernel to install > press Enter on the default selection.
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
  - Services to use > select all including "backported software"
- Select and install software
  - Updates management on this system > No automatic updates
  - Participate in the package usage survey > No
  - Choose software to install > Remove all selections other than leaving "SSH Server" (if installing for a server) and "Standard system utilities" selected
- Install the GRUB boot loader > Enter
  - Force GRUB installation to the EFI removable media path > No
  - Update NVRAM variables to automatically boot into Debian > Yes
  - Run os-prober automatically to detect and boot other OSes > No (if a dedicated system)
- Finish the installation > Enter
  - Is the system clock set to UTC > Yes
  - Reboot to Continue > Enter (remove usb)

## Post Reboot - Updates and ZRAM Configuration

- Login
- Execute the following commands.
  - sudo apt update
  - sudo apt upgrade (if anything)
  - sudo apt install zram-tools micro
  - lsblk   (to see amount of zram)
  - sudo micro /etc/default/zramswap
    - Uncomment **ALGRO=lz4**
    - Uncomment, and alter **PERCENT=25** (Depends on amount of RAM, ideally want around 8-10GB)  OR alter to a specific amount **SIZE=8192**
    - Ctrl + S, Ctrl + Q
  - sudo reboot
  - lsblk (to verify amount of zram post reboot)
  