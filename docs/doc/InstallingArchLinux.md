
# Installing Arch Linux

## Before We Begin

This guide assumes a uefi system.

* [Download Arch ISO](https://archlinux.org/download/)
* Write to USB With [Balena Etcher](https://www.balena.io/etcher/), [Rufus](https://rufus.ie/en/#), or [Ventoy](https://www.ventoy.net/en/download.html)  
* Boot to the [Live USB](https://en.wikipedia.org/wiki/Live_USB)

## Installing

### Set Time  

``` bash
timedatectl set-ntp true
```

### Partition with [`cfdisk`](https://www.geeksforgeeks.org/cfdisk-command-in-linux-with-examples/)

```bash
cfdisk /dev/sdx
```

Replace `/dev/sdx` with the device name of the drive you want to install to.
You can find the block devices with the command `lsblk`.

|Mount Point|   Partition    | Partition Type |  Recommended Sizes  |
----|
|`/mnt/boot`|`/dev/efi_part` |efi_system_part |*500MB*              |
|`[swap]`   |`/dev/swap_part`|Linux swap      |*More than 1GB*      |
|`/mnt`     |`/dev/root_part`|Linux-filesystem|*Remainder of device*|

Replace `efi_part`, `swap_part`, and `root_part` with actual partition names, you can see these with `lsblk`.

Please feel free to diverge from this simple partition table, it can be very benificial to separate the
`/home` folder. Swap is very optional [(Learn More)](https://wiki.archlinux.org/title/Swap).
If you understand partitioning you may use this partition table, but there are better ways to configure your system.

### Format the partitions

``` bash
mkfs.ext4 /dev/root_part
mkswap /dev/swap_part
mkfs.fat -F 32 /dev/efi_part
```

> You don't need to mkswap if you don't make a swap partition.

The above commands make filesytems on the partitions you formatted.

* root partition - `ext4` There's better ones for partitcular usecases but this is generally the defualt, general purpose filesystem.
* boot partition - fat32

### Mount partitions

``` bash
mount /dev/root_part /mnt
mount --mkdir /dev/efi_part /mnt/boot
swapon /dev/swap_part
```

### Install Needed Packages

``` bash
pacstrap /mnt base base-devel linux linux-firmware vim git
```

pacstrap allows installation of packages with pacman to another system.
We're installing packages to our root partition at mounted at `/mnt`.

#### The explanation of each package

* base
  * Not a package but a group of packages, these are the minimal packages to define a basic Arch Linux installation.
  * The packages within the group include, but are not limited to:
    * bash
    * coreutils
    * file
    * grep
    * tar
    * util-linux
    * glibc
    * gzip
    * pacman
* base-devel
  * Not a package but a group of packages
  * The packages within the group include, but are not limited to:
    * gcc
    * autoconf
    * automake
    * make
    * which
    * gettext
    * findutils
    * binutils
* linux
  * The operating system kernel, there are alternative versions of this kernel.
* linux-firmware
  * Firmware files for the Linux kernel
* vim
  * Vi IMproved, a *great* text editor.
* git
  * The fast distributed version control system

### Generate fstab

The `/etc/fstab` file is a file that tells the system what filesystems to mount on boot.

``` bash
genfstab -U /mnt >> /mnt/etc/fstab
```

`genfstab`, part of the `arch-install-utils` package, generates an fstab file so you don't have to write it yourself.

### [`chroot`](https://wiki.archlinux.org/title/Chroot) into the system

``` bash
arch-chroot /mnt
```

This means we're basically entering our system to finish setup, it's not bootable or functional yet so we chroot.

### Set Time and Locales

``` bash
ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
hwclock --systohc
```

> Region/City example: America/New_York [US/East]  

[Edit](https://wiki.archlinux.org/title/Textedit "Textedit")  `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed [locales](https://wiki.archlinux.org/title/Locale "Locale"). After that, run

To edit this file with `vim`:

1. Type `vim /etc/locale.gen`
2. Press `i` or the `insert` key to enter insert mode (This allows us to edit the file).
3. Use the arrow keys or "vim keys" to scroll down to the locale you need.
4. When on the line(s) you want, delete the `#` (hash) in the begining of the line.
5. When you're done editing you'll need to save the changes and exit the editor, to do this
press `esc` (escape key) to exit insert mode. Now we enter the write and quit commands,
type `:wq` and press enter to do this.
6. Now we're done.

``` bash
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
echo "[your_computerhostname]" > /etc/hostname
```

You can change the locale that your language is sent to.

### Setup Users  

Set root password with `passwd`.
Ensure your password is strong as this is the key to your entire system.
The root account has the most authority on the system.

Edit sudoers file with `EDITOR='vim' visudo`, find and uncomment  
`#%sudo ALL=(ALl) All`. So it should look like this:  

```txt
%sudo ALL=(ALL) ALL
```

Create a user account with `sudo` permissions by using

```bash
groupadd sudo
useradd -m -G wheel,sudo [your_username_here]
passwd [your_username_here]
```

`-m` creates a home directory for the user, `-G` allows the setting of groups for the user.
Learn about group permissions [here](https://el-wumbus.github.io/Linux.md/doc/LinuxFilePermissions)

> If it returns an error, try running `groupadd sudo`
  
### Install grub bootloader  

``` bash
pacman -Sy grub
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

If dual-booting run `pacman -S os-prober` and edit the grub config at `/etc/default/grub` and setting `GRUB_DISABLE_OS_PROBER=true` to false, `GRUB_DISABLE_OS_PROBER=false`

### Finishing up  

For internet connection you may want to install and enable network manager. You can do this with the following:

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

to exit the chroot by typing `exit` then unmount the mounted partitions using `umount /dev/sdx1` for all the mounted partitions.  

do a `reboot` to and restart into the new arch system.  

## Desktop environment installation

>You can do all of this while in the `arch-chroot`

log in as root with either [`su`](https://wiki.archlinux.org/title/su) or by exiting the current session and using the username `root`.
now install the required packages for a desktop environment, for example kde-plasma.

``` bash
pacman -Sy xorg plasma-desktop sddm alacritty firefox
systemctl enable sddm
```

You'll want to install video drivers next. If you are running an modern nvidia graphics card then follow the instructions below. Otherwise, go to the [Arch wiki](https://wiki.archlinux.org)and search for your drivers.

```bash
sudo pacman -S nvidia nvidia-settings nvidia-utils
```

Now on boot you should see a login screen, just log in with the user account you made earlier.

## Use the AUR

***only install AUR packages as a regular user, without using sudo.*** It will ask for sudo password during the process.  

There are two ways to use the arch user repository, manually and using an AUR helper. Do not do either as root, or with sudo!

### Manually

***git** must be installed*

``` bash
git clone [aur package git link]
cd ./[aur package name]
makepkg -si
```

### With an AUR helper

***git** must be installed*

```bash
git clone https://aur.archlinux.org/yay.git 
cd yay
makepkg -si
cd ..
rm -rf yay
```

Then to use it, use  

```bash
yay -S [package]
```

yay uses the same syntax that pacman does.
