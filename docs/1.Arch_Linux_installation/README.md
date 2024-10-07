# Arch Linux Installation

This section focuses on the installation of Arch from scratch,
using the [`linux-zen`](https://github.com/zen-kernel/zen-kernel/wiki/FAQ) kernel.
I will specify some of my personal preferences here,
for more info refer to the [Arch Wiki Installation Guide](https://wiki.archlinux.org/title/Installation_guide).

Table of contents:
<!-- TOC -->
- [Step 1: Boot](#step-1-boot)
- [Step 2: Partition table](#step-2-partition-table)
- [Step 3: Format the partitions and mount them](#step-3-format-the-partitions-and-mount-them)
- [Step 4: Install Arch](#step-4-install-arch)
- [Step 5: Chroot](#step-5-chroot)
  - [5.1 User management, hostname and network](#51-user-management-hostname-and-network)
  - [5.2 Time and localization](#52-time-and-localization)
  - [5.3 Grub](#53-grub)
  - [5.4 SSD Trim scheduling](#54-ssd-trim-scheduling)
  - [5.5 Reboot](#55-reboot)
- [Step 6: First boot - Networking and YaY](#step-6-first-boot---networking-and-yay)
<!-- /TOC -->

## Step 1: Boot

Boot into the USB installation media. You may have to fiddle with your EFI firmware.
In my case it was as simple as pressing <kbd>F12</kbd> to bring up a one-time
choice menu.

>[!NOTE]
> If you are using an external display on a laptop and you have multiple
> video outputs, chances are that not all of them will display the boot process.
> For example, my laptop does not output anything on the HDMI until the OS is running,
> so I had to plug the external display on the USB-C port or use the integrated one.

Once booted, load your keyboard map if needed:

```bash
loadkeys it
```

## Step 2: Partition table

Create the partition table as suggested by the guide. I'm reporting it here
to also show my suggestions about sizes and the partition types to be configured.

| Mount point | Size          | Partition type (fdisk number) |
|-------------|---------------|-------------------------------|
| /boot       | at least 500M | "EFI System" (1)              |
| N/A         | Half RAM      | "Linux Swap" (19)             |
| /           | you choose    | "Linux Root (x86_64)" (23)    |

This order allows to easily give the root mount point all the disk's space.

> [!WARNING]
> The `/boot` partition NEEDS to be that size, otherwise there's the risk of
> not being able to boot correctly as the initramfs and the linux images
> can be quite large.

> [!NOTE]
> I'm not entirely sure that the swap partition is actually needed, especially
> as my own PC has 16GB of RAM.
> I'm evaluating the idea of reserving some RAM space to a ZRAM partition instead.

Use `cfdisk /dev/<your-block-device>` (eg: `cfdisk /dev/nvme0n1`), delete all the
unneeded partitions and recreate the partition table.
If it's the first time you use this disk, remember to create a GPT partition
table first.

> [!TIP]
> You can add other partitions if you want/need (eg: the `/home` partition).
> This guide will assume that you created the partition table as described above.

## Step 3: Format the partitions and mount them

```bash
mkfs.fat -F 32 /dev/<your-device><partition1> # eg: /dev/nvme0n1p1
mkswap /dev/<your-device><partition2> # eg: /dev/nvme0n1p2
mkfs.ext4 /dev/<your-device><partition3> # eg: /dev/nvme0n1p3
mount /dev/<your-device><partition3> /mnt
mount --mkdir /dev/<your-device><partition1>
swapon /dev/<your-device><partition2>
```

> [!WARNING]
> The first 2 partitions must be formatted exactly as shown.

> [!TIP]
> For the root partition I chose Ext4 for the sake of simplicity,
> if you want you can use another one (for example [BTRFS](https://wiki.archlinux.org/title/Btrfs)).

## Step 4: Install Arch

You need an internet connection now. If you're using an Ethernet cable you're
probably fine, otherwise you can use `iwctl`'s interactive console.

> [!NOTE]
>
> See [the `iwctl` page on the Arch Wiki](https://wiki.archlinux.org/title/Iwd#iwctl)
> for more details about its usage.

Run this snippet to install the base system plus some basic packages:

```bash
pacstrap -K /mnt base base-devel chrony\
    efibootmgr git grub intel-ucode\
    linux-firmware linux-zen\
    networkmanager plymouth sudo vim
```

> [!NOTE]
> If you have an AMD CPU, replace the `intel-ucode` package
> with `amd-ucode`.

> [!WARNING]
> Again, this is my own customization, for example I chose
> NetworkManager to manage networks. You can customize this list to your liking.
> Keep in mind that the bare minimum would be
> `base efibootmgr grub linux linux-firmware`
> (you can replace `linux` with whatever kernel variant you prefer).

Also run this line to generate the `fstab` file:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

You have a functioning Arch Linux already! You can't boot into it yet though...

## Step 5: Chroot

In this step you will operate inside the actual Arch instance:

```bash
arch-chroot /mnt
```

### 5.1 User management, hostname and network

Set a root password, create your main user, which will be able to `sudo`,
and set the hostname.

```bash
passwd # set the root password
useradd -m -G wheel <username> -c <Full Name> # this user will be able to sudo
passwd <username> # set <username>'s password
EDITOR=vim visudo # uncomment this line: # %wheel ALL=(ALL:ALL) ALL
echo <desired-hostname> > /etc/hostname
systemctl enable NetworkManager
```

### 5.2 Time and localization

Set your timezone and setup the synchronization with an NTP server.

```bash
# replace <region>/<city> with whatever applies to you, for me it's Europe/Rome
ln -sf /usr/share/zoneinfo/<region>/<city> /etc/localtime
hwclock --systohc
vim /etc/chrony.conf
```

Change the pool in line 34 with something near you, see https://www.ntppool.org/en/
for the available zones and pools. I used `it.pool.ntp.org`.

That line will look like this:

```plaintext
pool it.pool.ntp.org iburst offline
```

> [!NOTE]
> I also added `offline` at the end of that line because my PC, being a laptop,
> has the possibility to be offline at boot. That options tells `chronyd`
> to not wait for the online status at boot, making it ever so slightly faster.

Next, enable the chrony daemon to make it run at boot and setup the system locale:

```bash
systemctl enable chronyd
vim /etc/locale.gen
```

Uncomment `en_US.UTF-8`. If you skip the US locale, some software can have issues.
You can uncomment other locales if you want to set your system to use other
languages and formats, for example I also uncommented `it_IT.UTF-8`.

Then run:

```bash
locale-gen # this will generate the needed files for the locales you uncommented
echo "KEYMAP=<a keymap>" > /etc/vconsole.conf # I put there "it"
echo "LANG=<your locale>" > /etc/locale.conf # I put there "it_IT.UTF-8
```

The last line will configure the system to use the locale you specified.
You need this even if you want the system to be configured
to use the `en_US.UTF-8` locale.

### 5.3 Grub

Time to make this Arch installation bootable. Run the following command:

```bash
vim /etc/mkinitcpio.conf
```

and edit the following parameter as described:
<!-- markdownlint-disable MD013 -->
```plaintext
HOOKS=(systemd autodetect microcode modconf kms keyboard keymap consolefont block filesystems)
```
<!-- markdownlint-enable MD013 -->

> [!NOTE]
> See https://wiki.archlinux.org/title/Mkinitcpio#Common_hooks for an explanation
> about these hooks.

After that, run:

```bash
vim /etc/default/grub
```

and edit the following parameters as described:

```plaintext
GRUB_TIMEOUT=0
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 nosgx quiet splash"
GRUB_TIMEOUT_STYLE=hidden
```

> [!WARNING]
> The two `TIMEOUT` parameters, as configured above, tell GRUB to not show
> *AT ALL* and boot immediately.
> This means that if you need to edit the boot parameters, you have to be quick
> when pressing <kbd>esc</kbd> after the EFI firmware has finished loading.
> If you don't desire this behaviour, you can skip those lines.

After that, run:

```bash
mkinitcpio -P
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchGRUB
```

### 5.4 SSD Trim scheduling

Enable TRIM to optimize SSD usage:

```bash
systemctl enable fstrim.timer
```

### 5.5 Reboot

If everything went fine, you can unmount everything and reboot.

```bash
exit # from the chroot env
umount /mnt/boot
umount /mnt
swapoff
reboot
```

Unplug the USB installation media.

Congrats, you have completed the installation of a bootable Arch Linux
with the `linux-zen` kernel! 🚀

## Step 6: First boot - Networking and YaY

After the first boot, make sure that you can connect to the Internet. If you have
a WiFi, run `nmtui` and setup your connection.

Then install [YaY](https://github.com/Jguer/yay?tab=readme-ov-file#binary)
to easily manage AUR packages:

```bash
sudo pacman -S git base-devel
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

[Go back to index](../#guides)

[Next: GNOME DE](../2.Gnome_DE/)
