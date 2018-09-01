# Enter BIOS with F2 and configure:
# - "System Configuration" > "SATA Operation": "AHCI"
# - "Secure Boot" > "Secure Boot Enable": "Disabled"
# - „Boot Mode“ Thourough

# Enter boot menu with F12, and boot the Arch USB medium

# Set desired keymap
loadkeys de-latin1

# Connect to Internet
wifi-menu

# Sync clock
timedatectl set-ntp true

# Create two partitions:
# 1 512MB EFI partition # Hex code ef00
# 2 100% Linux partiton (to be encrypted) # Hex code 8300
cfdisk /dev/nvme0n1

mkfs.fat -F32 /dev/nvme0n1p1

# Setup the encryption of the system
cryptsetup luksFormat /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 luks

# Create LVM partitions
# This creates partitions for root and /home, no /swap.
pvcreate /dev/mapper/luks
vgcreate vg0 /dev/mapper/luks
lvcreate -L 24G vg0 --name swap
lvcreate -L 25G vg0 --name root
lvcreate -l +80%FREE vg0 --name home	# 80% leaves some space for snapshots

mkfs.ext4 /dev/mapper/vg0-root
mkfs.ext4 /dev/mapper/vg0-home
mkswap /dev/mapper/vg0-swap

mount /dev/mapper/vg0-root /mnt
mkdir /mnt/home
mount /dev/mapper/vg0-home /mnt/home
swapon /dev/mapper/vg0-swap




mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

# Install the base system plus a few packages
pacstrap /mnt base zsh vim git sudo efibootmgr wpa_supplicant dialog iw

# Generate fstab
genfstab -L /mnt >> /mnt/etc/fstab

# Verify and adjust /mnt/etc/fstab
# For all non-boot partitions configure /mnt/etc/fstab:
# - Change "relatime" to "noatime" to reduce wear on SSD
# - perhaps more (discard, and autodefrag when using btrfs instead of LVM)

# Enter the new system
arch-chroot /mnt

# Setup time
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
hwclock --systohc

# Generate required locales
vi /etc/locale.gen	# Uncomment desired locales, e.g. "en_US.UTF-8", "de_CH.UTF-8"
locale-gen

# Set desired locale
echo 'LANG=en_US.UTF-8' > /etc/locale.conf

# Set desired keymap and font
echo 'KEYMAP=de-latin1' > /etc/vconsole.conf


# Set the hostname
echo '<hostname>' > /etc/hostname
# Add to /etc/hosts :
127.0.0.1	localhost
::1		localhost
127.0.1.1	<hostname>.localdomain	<hostname>

# Set password for root
passwd

# Add real user
useradd -m -g users -G wheel -s /bin/zsh <username>
passwd <username>
echo '<username> ALL=(ALL) ALL' > /etc/sudoers.d/<username>

# Configure mkinitcpio with modules needed for the initrd image
vi /etc/mkinitcpio.conf
# Add (ext4 dm_snapshot) to MODULES
# Change: HOOKS=(base systemd autodetect modconf block keyboard sd-vconsole sd-encrypt sd-lvm2 filesystems fsck)

# Regenerate initrd image
mkinitcpio -p linux

# Setup systemd-boot
bootctl --path=/boot install

# Enable Intel microcode updates
pacman -S intel-ucode

# Create bootloader entry
# Get luks-uuid with: `cryptsetup luksUUID /dev/nvme0n1p2`
---
/boot/loader/entries/arch.conf
---
title          Arch Linux
linux          /vmlinuz-linux
initrd		/intel-ucode.img
initrd         /initramfs-linux.img
options		luks.uuid=<uuid> luks.name=<uuid>=luks root=/dev/mapper/vg0-root rw
---

# Set default bootloader entry
---
/boot/loader/loader.conf
---
default		arch
---

# Exit and reboot
exit
reboot
