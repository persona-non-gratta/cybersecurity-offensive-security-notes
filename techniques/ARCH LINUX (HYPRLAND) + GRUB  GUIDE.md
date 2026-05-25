# Arch Linux Installation Notes

## Time Synchronization
```bash
timedatectl set-ntp true
```

Enables automatic time synchronization via NTP.  
Important for HTTPS, certificates, and package management.

---
# Disk Partitioning
```bash
fdisk -l #Shows all disks and partitions in the system.
fdisk /dev/... #Opens the partition editor for the selected disk.
g #Creates a new GPT partition table.  
n #Creates a new partition.
1 #Partition number.
+2G #Partition size.  Here, a 2 GB swap partition is created.
```
## Creating the Main Partition
```bash
n
2
```
Creates the second partition — **the main system partition.**
## Setting Partition Type
```bash
t #changes the type
1 #part.number
19 #linux swap type
```
Changes the first partition type to Linux Swap.

```bash
w
```
Writes changes to disk and exits `fdisk`.

```bash
fdisk -l
```
Checks the final partition layout.

---
# Swap and Filesystem
```bash
mkswap /dev/<swap partition> #Formats the partition as swap space.
swapon /dev/<swap partition> #Enables the swap partition.
```

```bash
mkfs.ext4 /dev/<fs partition>
```
Creates an ext4 filesystem on the main partition.

```bash
mount /dev/<fs partition> /mnt
```
Mounts the partition to `/mnt`.  
Arch Linux will be installed there.

```bash
lsblk
```
Displays mounted disks and partitions.  
You should see `[SWAP]` and `/mnt`.

---
# Installing the Base System
```bash
pacstrap /mnt base linux linux-firmware
```
Installs the base Arch system, Linux kernel, and firmware.

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Generates the filesystem table (`fstab`) automatically. **IMPORTANT!**

```bash
arch-chroot /mnt
```
Enters the installed system environment.

---
# Time Zone Setup
```bash
ln -sf /usr/share/zoneinfo/Europe/<location> /etc/localtime
```
Sets the system timezone.

```bash
hwclock --systohc
```
Synchronizes the hardware clock with system time.

---
# Installing Basic Utilities

```bash
pacman -S nano wget git curl sudo sh
```

**Installs basic tools:**
- `nano` — text editor
- `wget` / `curl` — downloading files
- `git` — Git support
- `sudo` — privileged command execution

---
# Locale Setup
```bash
nano /etc/locale.gen
```
Opens locale configuration.

Remove `#` from:
```text
en_US.UTF-8 UTF-8
ru_RU.UTF-8 UTF-8
```

Then generate locales:
```bash
locale-gen
```
Creates the selected language locales.

---
# Hostname Configuration
```bash
nano /etc/hostname
```
Sets the system hostname.

Adds local hostname mappings:
```bash
nano /etc/hosts
127.0.1.1   <system-name>.localdomain <system-name>
```
---
# Root Password
```bash
passwd
```
Sets the root password.

---
# User Creation
```bash
useradd -m <username>
```
Creates a new user with a home directory.

```bash
passwd <username>
```
Sets the user password.

```bash
usermod -aG wheel,audio,video,storage,vbox <username>
```

**Adds the user to important groups:**
- `wheel` — sudo access
- `audio` / `video` — hardware access
- `storage` — storage device access
- `vbox` — VirtualBox support

---
# Sudo Configuration

```bash
EDITOR=nano visudo
```
Opens the sudoers file safely.

Uncomment:
```text
%wheel ALL=(ALL) ALL
```
Allows users in the `wheel` group to use `sudo`.

---
# Network Setup

```bash
pacman -S networkmanager
```
Installs NetworkManager.

```bash
systemctl enable NetworkManager
```
Enables NetworkManager at boot.

---
# EFI Check

```bash
ls /sys/firmware/efi
```
Checks whether the system is booted in UEFI mode.

---
# GRUB Installation

```bash
pacman -S grub efibootmgr os-prober
```

Installs:
- `grub` — bootloader
- `efibootmgr` — EFI boot entry manager
- `os-prober` — detects other operating systems

```bash
nano /etc/default/grub
```
Opens GRUB configuration.

Uncomment:

```text
GRUB_DISABLE_OS_PROBER=false
```
Allows GRUB to detect Windows automatically.

```bash
mount | grep boot
```
Checks mounted boot partitions.

```bash
mount /dev/<EFI> /boot/efi
```
Mounts the EFI partition.

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```
Installs GRUB in UEFI mode.

### Alternative:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
Used depending on your EFI mount point.

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
Generates the GRUB configuration file.

If everything works correctly, you should see:
```text
Found Windows Boot Manager
```

```bash
reboot
```
Restarts the system.

---
# Hyprland Auto Installer

```bash
sh <(curl -L https://raw.githubusercontent.com/JaKooLit/Arch-Hyprland/main/auto-install.sh)
```

**Downloads and runs the Hyprland auto-install script from GitHub.** Auto-installation of:
1) Drivers
2) Hyprland
3) Dots
4) Useful Scripts
