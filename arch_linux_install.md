# Steps to Install Arch Linux

Obtain the `.iso.sig` and the `.iso` arch linux files in the same directory from https://archlinux.org/download/, then verify the signature:

```bash
pacman-key -v arch.iso.sig
```

Flash the `.iso` into a device, like `/dev/sdc`:

```bash
sudo cp arch.iso /dev/sdc
```

Boot the live environment.

[Connect to the internet.](https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet)

Setup the disk.

```bash
sgdisk -Z -n1:0:+1024M -t1:ef00 -c1:efi -n2:0:+4096M -t2:ef02 -c2:boot -N3 -t3:8309 -c3:root /dev/sda
```

Load the encryption modules.

```bash
modprobe dm-crypt && modprobe dm-mod
```

Set up the encryption and then open it.

```bash
cryptsetup luksFormat -s 512 -h sha512 /dev/sda3
cryptsetup open /dev/sda3 luks_lvm
```

Create the volume and volume group.

```bash
pvcreate /dev/mapper/luks_lvm
vgcreate arch /dev/mapper/luks_lvm
```

Create a volume for your swap space. A good size for this is your RAM size (find out with `free -h`) + 2GB.

```bash
lvcreate -n swap -L 18G arch
```

Use entire disk space for your root volume.

```bash
lvcreate -n root -l +100%FREE arch
```

Create filesystems

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.btrfs -L root /dev/mapper/arch-root
```

Setup swap device

```bash
mkswap /dev/mapper/arch-swap
swapon /dev/mapper/arch-swap
swapon -a
```

Mount Root, Boot and EFI

```bash
mkdir -p /mnt/boot /mnt/boot/efi
mount /dev/mapper/arch-root /mnt
mount /dev/sda2 /mnt/boot
mount /dev/sda1 /mnt/boot/efi
```

Install Arch

```bash
pacstrap -K /mnt base sof-firmware base-devel linux linux-firmware neovim btrfs-progs lvm2 grub efibootmgr zsh
```

Load the file table and chroot.

```bash
genfstab -U -p /mnt > /mnt/etc/fstab
arch-chroot /mnt /bin/bash
```

Add hooks.

```bash
sudo sed -i '/^HOOKS=.*block/s/block /block encrypt lvm2 /' /etc/mkinitcpio.conf
```

Setup grub on efi partition

```bash
grub-install --efi-directory=/boot/efi
```

Add cryptdevice to linux commandline arguments

```bash
sed -i '/^GRUB_CMDLINE_LINUX_DEFAULT=/ s/"$/ root=\/dev\/mapper\/arch-root cryptdevice=UUID='$(blkid -s UUID -o value /dev/sda3)':luks_lvm"/' /etc/default/grub
```

```bash
mkdir /secure
dd if=/dev/random of=/secure/root_keyfile.bin bs=512 count=8
```

Change permissions

```bash
chmod 000 /secure/*
chmod 600 /boot/initramfs*
```

Add to partitions

```bash
cryptsetup luksAddKey /dev/sda3 /secure/root_keyfile.bin
```

Recognize root keyfile

```bash
sed -i 's/FILES=()/FILES=(\/secure\/root_keyfile.bin)/' your_file
```

Reload linux

```bash
mkinitcpio -p linux
```

Create grub config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
```

Locale

```bash
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

NTP

```bash
echo "[Time]\nNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org\nFallbackNTP=0.pool.ntp.org 1.pool.ntp.org" > /etc/systemd/timesyncd.conf
```

Enable timesyncd

```bash
systemctl enable systemd-timesyncd.service
```

Network manager for wifi

```bash
pacman -S networkmanager
systemctl enable NetworkManager.service
```

Locale

```bash
sed -i -e "/^#"en_GB.UTF-8"/s/^#//" /mnt/etc/locale.gen
echo "KEYMAP=us" > /etc/vconsole.conf
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
locale-gen
```

Hostname

```bash
echo "arch" > /etc/hostname
```

First secure the root user by setting a password

```bash
passwd
```

Add a new user as follows

```bash
useradd -m -k /var/empty -G wheel -s /bin/zsh e
passwd e
```

Add the wheel group to sudoers

```bash
echo "%wheel ALL=(ALL:ALL) ALL" > /etc/sudoers.d/wheel
```

Install amd or intel microcode depending on which processor you use (`lscpu`):

```sh
pacman -S amd-ucode # or intel-ucode
```

```bash
exit
umount -R /mnt
reboot 
```

Put UEFI Secure Boot into "Setup Mode".

```bash
sudo sbctl create-keys
sudo sbctl enroll-keys -m
```
