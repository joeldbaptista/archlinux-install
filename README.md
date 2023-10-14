# How to install archlinux

## Burn ISO into USB

```Bash
$ sudo dd if=/path/to/archlinux.iso do=/dev/sdX bs=1M
```
Where `/dev/sdX` is the USB you want to burn.

## Recipe

1. **Set your keyboard layout**.  

If your keyboard is not `US`, the default, you'll need to change the it. This can be achieved by running this:

```Bash
$ loadkeys pt-latin1
```
In the example, we're setting the keyboard layout to Portuguese. You can search for your layout by running:

```Bash
$ ls /usr/share/kbd/keymaps/**/*.map.gz | grep <search-key>
```

In `search-key` you add your country's initials (`fr` for France, `de` for Germany, etc). Use the name of the file as argument for `loadkeys`.

2. **Check your system's boot mode**

```Bash
$ ls /sys/firmware/efi/efivars
```
If you get a list of stuff, your system boots in `UEFI` mode. Otherwise, it boots in `BIOS`.

3. **Connect to internet**

Run:

```
$ iwct
```

Then, `device list` to list the available interfaces. Pick the right one - probably there's only one. Then, `station device scan` scan for networks. 
Finally, `station device get-networks` to list the `SSID`s, and `station device connect SSID`. And `exit` to quit. 

Alternatively, do:

```bash
$ iwctl station WIFI_INTERFACE connect "WIFI_ESSID" --passphrase "WIFI_KEY"
```
Where WIFI_INTERFACE is the wifi interface (e.g. wlan0), WIFI_ESSID the name of the network, and WIFI_KEY it's password. 

Test: `ping archlinux.org`

4. **Update the system clock**

Do `timedatectl set-ntp true`, and validate it with `timedatectl status`.

5. **Partition the disks**

**FOR BIOS** have a look at [here](https://www.itzgeek.com/how-tos/linux/arch-linux/install-arch-linux-2021.html).

Do `lsblk` to list the disks. Then `gdisk /dev/sdX`, where `/dev/sdX` is the disk where we're going to install Arch. Next, in the command line dialog, select `x` for expert, and then `z` to wipe out the disk. Just answer `Y` next, and next.

Next, do `cgdisk /dev/sdX`. This will open a ncurses form. Create the partitions you wish. Here we'll create a boot partitions (code `EF00` because my system boots `UEFI`, `EFO2` for `BIOS`). I usually put around 256M for this partition. Next, I create a swap partition (code `8200`). Finally, the rest is for root (code `8300`). Some people create more partions (`home`, `root`, `var`, etc). I haven't got the time for that.  

For swap size, there's a lot of info on the webs that you may use to decide how much swap we really need. Usually it's function of the amount of RAM you have available. Something like "1.5 x RAM". 

6. **Format the partitions**

For `boot`, do `mkfs.fat -F32 /dev/sdX1` (if the system is UEFI; if the system is BIOS, **do not** format), and for `root` do `mkfs.ext4 /dev/sdX3`. Swap is a little different:

```Bash
$ mkswap /dev/sdX2 && swapon /dev/sdX2
```

7. **Mount the file systems**

Do:

```Bash
$ mount /dev/sdX3 /mnt         # Mount root 
$ mkdir /mnt/boot              # Create a boot dir
$ mount /dev/sdX1 /mnt/boot    # Mount boot
```
If other partitions were created, it would be necessary to create directories for each one of them, and the mount them.

8. **Install base system**

Do: 

```Bash
$ pacstrap /mnt base base-devel linux linux-firmware vi networkmanager openssh
```

You can add more packages to install, or install later. If you get trust errors when pacstraping, update the keyring before running pactrap, with this
command:

`pacman -Sy archlinux-keyring`

9. **Generate ftab**

Do `genfstab -U /mnt >> /mnt/etc/fstab`.

10. **Configure the system**

Do `arch-chroot /mnt` to get in the newly installed system. And perform the necessay configurations, such as:

```Bash
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime ## Set timezone
$ hwclock --systohc ## Set hardware clock
$ nvim /etc/locale.gen ## And uncomment your locale
$ locale-gen
$ echo LANG=en_IE.UTF-8 > /etc/locale.conf  ## The encoding you've picked
$ echo KEYMAP=pt-latin1 > /etc/vconsole.conf ## The encoding you've picked before
$ echo lekonne > /etc/hostname ## Name your machine
$ mkinitcpio -P ## The guide claims this is not necessary... 
$ passwd ## Set root password
$ useradd -m -g users -G wheel,storage,power,network,docker -s /bin/bash dasuser ## Create a your user
$ passwd dasuser ## Change password for dasuser
$ EDITOR=nvim visudo ## Configure wheel users; personally I set NOPASSWD for wheel users (me)
```

11. **Install bootloader**

Do `bootctl install`, next `nvim /boot/loader/entries/arch.conf` and write:

```Bash
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img ## this if you have intel CPU; if not, consult the installation guide for more details
initrd /initramfs-linux.img
```
Next, `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sdX) rw" >> /boot/loader/entries/arch.conf` where `/dev/sdX` is the root partition.
Next, `pacman -S intel-ucode` if you've got an intel CPU. 

12. **Prepare Network Manager wifi service & al**

Do `systemctl enable NetworkManager`, `systemctl enable sshd` and `systemctl enable docker`. And `exit`, and `umount -R /mnt`. Finally, reboot.

## References

[Archlinux installation guide](https://wiki.archlinux.org/title/Installation_guide)  
[iwctl archlinux documentation](https://wiki.archlinux.org/title/Iwd)  

