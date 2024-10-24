# Useful shell commands

Installing `.pkg.tar.zst` file with `sudo pacman -U my-package.pkg.tar.zst`

Flash iso file on a drive e.g. `/dev/sdc`:

```
sudo cp path/to.iso /dev/sdc
```

### Debugging

Some useful commands:

- `ip link`: display state of network interfaces
- `dmesg`: shows kernel messages
- `lspci -k`: display information about each pci bus connected to the system
- `journalctl -b -1 -e`: info about the last second before the previous shutdown

### How to install a package on an offline system

```bash
sudo mkdir /var/cache/pacman/custom-pkgs
sudo pacman -Sw --cachedir /var/cache/pacman/custom-pkgs <your-package>
```

Then transfer the `*.pkg.tar.zst` files to the offline device

On the offline device:
```bash
cp /mount_of_usb_drive/ *.pkg.tar.zst /var/cache/pacman/pkg/
```

Then:

```bash
pacman -U --asdeps /var/cache/pacman/pkg/*.tar.zst
```
