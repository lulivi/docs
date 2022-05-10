# Archlinux installation guide

- [Pre-installation](#pre-installation)
  - [Connect to internet](#connect-to-internet)
  - [Partitions](#partitions)
  - [Setup partitions](#setup-partitions)
  - [Mount partitions and create swapfile](#mount-partitions-and-create-swapfile)
  - [Install base and enter system](#install-base-and-enter-system)
- [Config System](#config-system)
  - [SSD](#ssd)
  - [Locale](#locale)
  - [Keyboard](#keyboard)
  - [Hostname](#hostname)
  - [Users](#users)
  - [Boot](#boot)
- [Post-installation](#post-installation)
  - [UI](#ui)
  - [Audio](#audio)
  - [Programs](#programs)
  - [Utils](#utils)

## Pre-installation

### Connect to internet

Check interface with `ip link`. Then `dhcpcd <interface>`. Test the connection
with `ping`.

Also checkout *evi vars* are present in `/sys/firmware/efi/efivars` for UEFI
system installations.

### Partitions

Using `cfdisk /dev/sda`, create two partitions:

| Partition | Size      | Type       |
|:---------:|:---------:|:----------:|
| /dev/sda1 | 512M      | Efi System |
| /dev/sda2 | Remaining | Linux LVM  |

### Setup partitions

We will use LVM tools:

```bash
pvcreate /dev/sda2
vgcreate /dev/sda2 vg
```

Then create logic volumes:

```bash
lvcreate -L 80G vg -n root
lvcreate -l 100%FREE vg -n home
```

And format them:

```bash
mkfs.fat -f32 /dev/sda1
mkfs.ext4 /dev/vg/root
mkfs.ext4 /dev/vg/home
```

### Mount partitions and create swapfile

Firstly mount root partition:

```bash
mount /dev/vg/root /mnt
```

After that, create swapfile and mount it:

```bash
fallocate -l 8G /mnt/swapfile
chmod 600 /mnt/swapfile
mkswap /mnt/swapfile
swapon /mnt/swapfile
```

Then mount the other partitions:

```bash
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
mkdir /mnt/home
mount /dev/vg/home /mnt/home
```

### Install base and enter system

Install base system:

```bash
pacstrab /mnt base base-devel
```

Generate fstab:

```bash
genfstab -U -p /mnt > /mnt/etc/fstab
```

And modify it to fix swap path:

```plaintext
>>> /mnt/etc/fstab
- /mnt/swapfile none swap defaults 0 0
+ /swapfile none swap defaults 0 0
```

Enter the system:

```bash
arch-chroot /mnt
```

## Config System

### SSD

If using SSD:

```bash
systemctl enable fstrim.timer
```

### Locale

Set localtime:

```bash
ln -sf /usr/share/zoneinfo/europe/Madrid /etc/localtime
hwclock --systohc
```

Remove comment in desired line inside `/etc/locale.gen`. For example `#
en_US-UTF-8`. Then run:

```bash
locale-gen
```

### Keyboard

To configure keyboard for the current session:

```bash
setxkbmap -model <model> \
          -layout <layout1>[,<layout2>] \
          -variant <variant1>[,<variant2>] \
          -option <option1> -option <option2>
```

e.g.:

```bash
setxkbmap -model pc104 -layout us,us -variant altgr-intl,dvorak-alt-intl -option grp:alt_space_to
setxkbmap -model pc104 -layout us -variant altgr-intl -option ctrl:nocaps
```

This will set the keyboard to US layout with dead keys using AltGr (áéíóúñ).
Also an alternative layout with Dvorak can be switched using `Alt`+`Space`.

To make this configuration default after restart, we will use `localectl`:

```bash
localectl set-x11-keymap <layout1>[,<layout2>] \
                         <variant1>[,<variant2>] \
                         <option1>[,<option2>]
```

e.g.:

```bash
localectl set-x11-keymap us,us pc104 altgr-intl,dvorak-alt-intl grp:alt_space_toggle
localectl set-x11-keymap us pc104 altgr-intl ctrl:nocaps
```

Some util options:

| Option          | Comment                 |
|-----------------|-------------------------|
| `ctrl:nocaps`   | Caps Lock as Ctrl       |
| `ctrl:swapcaps` | Swap Ctrl and Caps Lock |

### Hostname

And hostname config:

```bash
echo "bug" > /etc/hostname
```

```plaintext
>>>/etc/hosts
+ 127.0.0.1  localhost
+ ::1        localhost
+ 127.0.1.1  bug.localdomain bug
```

### Users

Set root password with `passwd`.

Create user:

```bash
useradd -m -g users -G wheel,storage,power -s /bin/bash <username>
passwd <username>
```

Allow new user to use sudo commands with sudo password:

- Uncomment `# %wheel ALL=(ALL) ALL` line
- Add `Defaults rootpw`

### Boot

Add lvm to initcpio:

```plaintext
>>>/etc/mkinitcpio.conf
- HOOKS="...block filesystems..."
+ HOOKS="...block llvm2 filesystems..."
```

Create initial ramdisk environment:

```bash
cd /boot
mkinitcpio -p
```

Mount efivars and install bootloather (for example systemd):

```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
bootctl install
```

Then, modify loader and loader entry:

```plaintext
>>>/boot/loader/loader.conf
+ default arch
+ timeout 5
```

```plaintext
>>>/boot/loader/entries/arch.conf
title     Arch Linux
linux     /vmlinuz-linux
initrd    /initramfs-linux.img
options   root=/dev/mapper/<volume-group-name>-<root-logic-volume> rw

If you have Intel, install microcode updates:

```bash
pacman -S intel-ucode
```

And modify arch entry:

```plaintext
>>>/boot/loader/entries/arch.conf
+ initrd    /intel-ucode.img
```

If you have Nvidia uncompatibilities with nouveau driver:

```plaintext
>>>/boot/loader/entries/arch.conf
- options   root=...
+ options   root=... modprobe.blacklist=nouveau
```

Finalizing instalation:

```bash
exit
umount -R /mnt
reboot
```

## Post-installation

Missing modues (`aic94xx` and `wd719x`). Install them from AUR:

- [aic94xx](https://aur.archlinux.org/packages/?K=aic94xx)
- [wd719x](https://aur.archlinux.org/packages/?K=wd719x)

```bash
git clone <module-url>
cd <module-name>
less PKGBUILD
makepkg -si
```

For wireless network install broadcom dkms:

```bash
pacman -S broadcom-wl-dkms
```

### UI

Install xorg and intel drivers (if needed)

```bash
pacman -S xorg xf86-video-intel
```

Then install DE and DM:

```bash
pacman -S xfce4 lxdm
systemctl enable xldm
systemctl start lxdm
```

### Audio

Install audio utils:

```bash
sudo pacman -S alsa-utils pulseaudio pavucontrol xfce4-pulseaudio-plugin
```

### Programs

```bash
sudo pacman -S firefox \ # browser
               st \ # + https://st.suckless.org/patches/spoiler/
               mousepad \ # text editor
               pcmanfm file-roller \ # file manager
               evince \ # pdf viewer
               feh \ # ?
               ristretto \ # image viewer
               rofi # TODO: window switcher
```

### Utils

```bash
sudo pacman -S acpilight \ # screen brightness
               udiskie \ # auto mount removal media
```

