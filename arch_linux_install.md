# Steps to Install Arch Linux

## Creating a Bootable USB Drive

### Download

1. Download the `.iso`, `.iso.sig`, `b2sums.txt`, and `sha256sums.txt` files from the Arch Linux install page (Arch Linux > Download > Worldwide > geo.mirror.pkgbuild.com).

### Security

#### Verify the Hash Sum of the Image:

```bash
# In the same folder as the .iso
# Expected output: OK
b2sum -c b2sums.txt --ignore-missing

# Expected output: OK
sha256sum -c sha256sums.txt --ignore-missing

# Download and import the public key to verify the signature
gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org
gpg --verify archlinux.iso.sig
```

### Flashing

Just use `cp` (assuming you want to flash to `/dev/sdc`):

```
sudo cp path/to.iso /dev/sdc
```

## Installing Arch

Boot from the USB stick into the Arch ISO installer.

### Keyboard Layout (Optional)

```bash
localectl list-keymaps # Find all keyboard layouts.
loadkeys us # This is the default.
```

### Setting Up an Internet Connection

#### Connecting via Ethernet Cable

Plug an ethernet cable in; the internet should work out of the box.

#### Connecting via Wi-Fi

```bash
iwctl
device list # Choose the station/device name to connect to, e.g., `foo`, and the Wi-Fi name `bar`
station foo connect bar

# Verify internet connection, ensure you receive responses before proceeding
ping -c 4 archlinux.org
exit
```

### Setting Up the Partitions

Get the names of the blocks

```
$ lsblk
```

For both partition setups, you'll want to setup a table on your primary drive.

```
$ gdisk /dev/block_name
```

Inside of gdisk, you can print the table using the `p` command.

To create a new partition use the `n` command. `d` to delete. The below table shows
the disk setup I have for my primary drive

| partition | first sector | last sector | code |
| --------- | ------------ | ----------- | ---- |
| 1 (efi)   | default      | +512M       | ef00 |
| 2 (boot)  | default      | +4G         | ef02 |
| 3 (root)  | default      | default     | 8309 |

If you have a second drive for your home disk, then your table would be as
follows.

| partition | first sector | last sector | code |
| --------- | ------------ | ----------- | ---- |
| 1 (home)  | default      | default     | 8302 |

When done setting up the partitions, `w` to write.

#### Encrypt the Partitions

Make sure the encryption modules are loaded.

```bash
modprobe dm-crypt
modprobe dm-mod
```

Setting up encryption on our luks lvm partiton

```bash
cryptsetup luksFormat -v -s 512 -h sha512 /dev/a...3 # root
```

If you have a home partition:

```bash
cryptsetup luksFormat -v -s 512 -h sha512 /dev/b...1 # home
```

Mount the drives:

```bash
cryptsetup open /dev/a...3 luks_lvm # root
```

If you have a home parition:

```bash
cryptsetup open /dev/b...1 arch-home # home
```

### Volume setup

Create the volume and volume group

```bash
pvcreate /dev/mapper/luks_lvm

vgcreate arch /dev/mapper/luks_lvm
```

Create a volume for your swap space. A good size for this is your RAM size (find out with `free -h`) + 2GB.

```bash
lvcreate -n swap -L 18G arch
```

Next you have a few options depending on your setup

### Single Disk

If you have a single disk, you can either have a single volume for your root
and home, or two separate volumes.

#### No home partition

Single volume is the most straightforward. To do this, just use the entire
disk space for your root volume

```bash
lvcreate -n root -l +100%FREE arch
```

#### With home partition

For two volumes, you'll need to estimate the max size you want for either your
root or home volumes. With a root volume of 200G, this looks like:

```bash
lvcreate -n root -L 200G arch
```

Then use remaining disk space for home

```bash
lvcreate -n home -l +100%FREE arch
```

## Filesystems

FAT32 on EFI partiton

```bash
mkfs.fat -F32 /dev/a...1
```

EXT4 on Boot partiton

```bash
mkfs.ext4 /dev/a...2
```

BTRFS on root

```bash
mkfs.btrfs -L root /dev/mapper/arch-root
```

BTRFS on home if exists

```bash
mkfs.btrfs -L home /dev/mapper/arch-home
```

Setup swap device

```bash
mkswap /dev/mapper/arch-swap
```

### Mounting

Mount swap

```bash
swapon /dev/mapper/arch-swap
swapon -a
```

Mount root

```bash
mount /dev/mapper/arch-root /mnt
```

Create boot

```bash
mkdir -p /mnt/boot
```

If you have a home:

```bash
mkdir -p /mnt/home
```

Mount the boot partiton

```bash
mount /dev/a...2 /mnt/boot
```

Mount the home partition if you have one, otherwise skip this

```bash
mount /dev/mapper/arch-home /mnt/home
```

Create the efi directory

```bash
mkdir /mnt/boot/efi
```

Mount the EFI directory

```bash
mount /dev/a...1 /mnt/boot/efi
```

### Install arch

```bash
pacstrap -K /mnt base base-devel linux linux-firmware neovim btrfs-progs lvm2 grub efibootmgr zsh sof-firmware
```

Load the file table

```bash
genfstab -U -p /mnt > /mnt/etc/fstab
```

chroot into your installation

```bash
arch-chroot /mnt /bin/bash
```

## Configuring

### Decrypting volumes

Add `encrypt` and `lvm2` into the hooks array between `block` and `filesystems` in `/etc/mkinitcpio.conf`:

### Bootloader

Setup grub on efi partition

```bash
grub-install --efi-directory=/boot/efi
```

```bash
nvim /etc/default/grub
```

Append the following kernel parameters to the env variable `GRUB_CMDLINE_LINUX_DEFAULT`
You can fill in the `<uuid>` by typing `:r !blkid /dev/a...3`

```bash
root=/dev/mapper/arch-root cryptdevice=UUID=<uuid>:luks_lvm
```

### Keyfile

```bash
mkdir /secure
```

Root keyfile

```bash
dd if=/dev/random of=/secure/root_keyfile.bin bs=512 count=8
```

Home keyfile if home partition exists

```bash
dd if=/dev/random of=/secure/home_keyfile.bin bs=512 count=8
```

Change permissions on these

```bash
chmod 000 /secure/*
chmod 600 /boot/initramfs-linux*
```

Add to partitions

```bash
cryptsetup luksAddKey /dev/a...3 /secure/root_keyfile.bin

# skip below if using single disk
cryptsetup luksAddKey /dev/b...1 /secure/home_keyfile.bin
```

```bash
nvim /etc/mkinitcpio.conf
```

Add this:

```bash
FILES=(/secure/root_keyfile.bin)
```

### Home Partition Crypttab (Skip if single disk)

Open up the crypt table.

```bash
nvim /etc/crypttab
```

Add in the following line at the bottom of the table
You can get `<uuid>` by `:r !blkid /dev/b...1`

```bash
arch-home      UUID=<uuid>    /secure/home_keyfile.bin
```

### Grub

Reload linux

```bash
mkinitcpio -p linux
```

Create grub config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
```

### System Configuration

#### Timezone

```bash
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

#### NTP

```bash
nvim /etc/systemd/timesyncd.conf
```

Add in the NTP servers

```bash
[Time]
NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org
```

Enable timesyncd

```bash
systemctl enable systemd-timesyncd.service
```

Network manager

```bash
pacman -S networkmanager && systemctl enable NetworkManager.service
```

#### Locale

Uncomment the UTF8 lang you want:

```bash
nvim /etc/locale.gen
```

```bash
locale-gen
```

If you changed your keyboard layout:

```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```

```bash
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
```

#### hostname

```bash
echo "arch" > /etc/hostname
```

#### Users

First secure the root user by setting a password

```bash
passwd
```

Add a new user as follows

```bash
useradd -m -k /usr/share/misc -G wheel -s /bin/zsh nikita
```

set the password on the user

```bash
passwd nikita
```

Add the wheel group to sudoers

```bash
EDITOR=nvim visudo
```

Uncomment this line:

```bash
%wheel ALL=(ALL:ALL) ALL
```

### Reflector

Edit `/etc/xdg/reflector/reflector.conf` to use this config:

```sh
--save /etc/pacman.d/mirrorlist
--protocol https
--country "United Kingdom"
--latest 5
--sort rate
```
Enable reflector service:

```sh
systemctl enable --now reflector.service
systemctl enable --now reflector.timer
```

### Microcode

Install amd or intel microcode depending on which processor you use (find out with `lscpu`):

```sh
pacman -S amd-ucode # or intel-ucode
```

### Firewall

```bash
pacman -S ufw
```

```bash
systemctl enable --now ufw.service
ufw enable
ufw limit SSH
ufw logging off
```

### Trim

If you are using SSD, setup trim

```bash
systemctl enable fstrim.timer
```

### Secure boot

Before creating new keys and modifying EFI variables, it is advisable to backup the current variables, so that they may be restored in case of error.

```bash
pacman -S efitools && for var in PK KEK db dbx ; do efi-readvar -v $var -o old_${var}.esl ; done
```

```bash
exit
umount -R /mnt
reboot now
```

Now reboot into `UEFI` and put secure boot into **SETUP MODE**. Refer to your motherboard manufaturer's guide on how to do that.

For most systems, you can do this by, just going into **BOOT** tab, **enabling secure boot**, go to **SECURITY** tab and do **Erase all secure boot settings**.

Now save changes and exit.

After enabling Setup Mode, reboot again, which allows the system to clear any previous Secure Boot keys.

Now when booting into **Arch Linux** you'll be prompted to enter the passphrase to your LUKS partition.

Enter it and boot into the system. Login as **root**.

```bash
pacman -S sbctl
```

Check status

```bash
sbctl status
```

You should see that sbctl is not installed and secure boot is disabled.

```bash
sbctl create-keys
sbctl enroll-keys -m
```

Check status again

```bash
sbctl status
```

sbctl should be installed now, but secure boot will not work until the boot files have been signed with the keys you just created. 

```bash
sbctl verify
```

Now sign all the unsigned files. Usually the kernel and the boot loader need to be signed. For example: 

```bash
sbctl sign -s /boot/vmlinuz-linux
sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
# ...
```

```bash
mkinitcpio -P
```
