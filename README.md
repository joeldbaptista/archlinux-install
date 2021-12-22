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

Test: `ping archlinux.org`

4. **Update the system clock**

Do `timedatectl set-ntp true`, and validate it with `timedatectl status`.

5. **Partition the disks**

Do `lsblk` to list the disks. Then `gdisk /dev/sdX`, where `/dev/sdX` is the disk where we're going to install Arch. Next, in the command line dialog, select `x` for expert, and then `z` to wipe out the disk. Just answer `Y` next, and next.

Next, do `cgdisk /dev/sdX`. This will open a ncurses form. Create the partitions you wish. Here we'll create a boot partitions (code `EF00` because my system boots `UEFI`, `EFO2` for `BIOS`). I usually put around 256M for this partition. Next, I create a swap partition (code `8200`). Finally, the rest is for root (code `8300`). Some people create more partions (`home`, `root`, `var`, etc). I haven't got the time for that.  

For swap size, there's a lot of info on the webs that you may use to decide how much swap we really need. Usually it's function of the amount of RAM you have available. Something like "1.5 x RAM". 

6. **Format the partitions**

For `boot`, do `mkfs.fat -F32 /dev/sdX1`, and for `root` do `mkfs.ext4 /dev/sdX3`. Swap is a little different:

```Bash
$ mkswap /dev/sdX2 && swapon /dev/sdX2
```

7. **Mount the file systems**

```Bash
$ mount /dev/sdX3 /mnt         # Mount root 
$ mkdir /mnt/boot              # Create a boot dir
$ mount /dev/sdX2 /mnt/boot    # Mount boot
```

## References

[Archlinux installation guide](https://wiki.archlinux.org/title/Installation_guide)  
[iwctl archlinux documentation](https://wiki.archlinux.org/title/Iwd)  

