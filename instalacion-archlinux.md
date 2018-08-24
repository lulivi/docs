<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:1 -->

1. [Preparación](#preparacin)
	1. [Teclado](#teclado)
	2. [Conexión a internet](#conexin-a-internet)
	3. [Asegurarnos de tener sistema EFI](#asegurarnos-de-tener-sistema-efi)
2. [Partitcionado](#partitcionado)
	1. [Creamos la particion lvm](#creamos-la-particion-lvm)
	2. [Creamos un volumen físicio para la partición lvm](#creamos-un-volumen-fsicio-para-la-particin-lvm)
	3. [Asignamos al Asignamos grupo de volúmenes al volumen creado](#asignamos-al-asignamos-grupo-de-volmenes-al-volumen-creado)
	4. [Creamos las particiones logicas](#creamos-las-particiones-logicas)
	5. [Formateo de los volúmenes](#formateo-de-los-volmenes)
	6. [Los formateamos](#los-formateamos)
3. [Montado de volúmenes e instalación](#montado-de-volmenes-e-instalacin)
	1. [Instalación](#instalacin)
	2. [Configuracióñ del sistema](#configuraci-del-sistema)
		1. [Generar fstab](#generar-fstab)
		2. [Entrar en el sistema](#entrar-en-el-sistema)
		3. [Hacer que el sistema reconozca LVM](#hacer-que-el-sistema-reconozca-lvm)
		4. [TimeZone](#timezone)
		5. [Locale](#locale)
		6. [Nombre del equipo](#nombre-del-equipo)
		7. [Soporte TRIM para SSDs](#soporte-trim-para-ssds)
		8. [Añadir usuario](#aadir-usuario)
		9. [Configurar mkinitcpio para que nos pille LVM](#configurar-mkinitcpio-para-que-nos-pille-lvm)
		10. [Bootloader](#bootloader)
		11. [Finalizar Instalación básica](#finalizar-instalacin-bsica)
4. [Post instalación del sistema](#post-instalacin-del-sistema)
	1. [Conexión a internet](#conexin-a-internet)
	2. [Interfaz gráfica](#interfaz-grfica)
	3. [Configuración del sonido](#configuracin-del-sonido)
5. [Instalación de paquetes útiles](#instalacin-de-paquetes-tiles)
	1. [Paquetes útiles](#paquetes-tiles)
	2. [Posibles firmwares](#posibles-firmwares)

<!-- /TOC -->

# Preparación

## Teclado

    # loadkeys es

## Conexión a internet

Observaos la interfaz cableada con `ip link`

    # ip link
    ...
    2: enp2s0f0... # cableado
    3: wlp3s0bl... # inalambrico

Luego, si no tenemos acceso a internet levantamos la interfaz cableada

    # dhcpcd <interfaz>
    # ping archlinux.org

## Asegurarnos de tener sistema EFI

    # ls /sys/firmware/efi/efivars

# Partitcionado

## Creamos la particion física

Si existe la partición EFI System se usa esa partición para el `/boot`.

    # cgdisk /dev/sda
    > NEW
    > DEFAULT START SECTOR
    > DEFAULT END SECTOR
    > 8E00
    > Linux Filesystem
    > WRITE

## Inicializamos el volumen para que use lvm

    # pvcreate <particion>

Ejemplo

    # pvcreate /dev/sda5

## Le asignamos un grupo de volumenes al volumen físico inicializado anteriormente

    # vgcreate <grupo_volumenes> <volumen_fisico>

Ejemplo

    # vgcreate vg /dev/sda5

## Creamos las particiones logicas

    # lvcreate -L <tamaño> <nombre_grupo_volumenes> -n <nombre_volumen>

Ejemplo

    # lvcreate -L 30G vg -n root
    # lvcreate -L 6G vg -n swap
    # lvcreate -l 100%FREE vg -n home

## Formateo de los volúmenes

Si no existieran volumenes logicos bajo `/dev/mapper` ejecutamos lo siguiente

    # modprobe dm-mod
    # vgscan
    # vgchange -ay

Después, formateamos los volúmenes lógicos creados anteriormente

    # mkfs.ext4 /dev/mapper/vg-root
    # mkfs.ext4 /dev/mapper/vg-home
    # mkswap /dev/mapper/vg-swap

## Montado de volúmenes

Montamos la swap y el root

    # swapon /dev/mapper/vg-swap
    # mount /dev/mapper/vg-root /mnt

Para el boot miramos cual es la partición EFI y creamos una carpeta en /mnt para montarla

    # mkdir /mnt/boot
    # mount /dev/sda2 /mnt/boot

Y finalmente el home

    # mkdir /mnt/home
    # mount /dev/mapper/vg-home /mnt/home

# Instalación

Instalamos los paquetes básicos de nuestro sistema

    # pacstrap /mnt base base-devel

Y generamos el archivo `fstab` para el arranque

    # genfstab -U -p /mnt > /mnt/etc/fstab

## Entramos en el nuevo sistema y lo configuramos

    # arch-chroot /mtn

### Hacer que el sistema reconozca LVM

    # echo "USELVM=\"yes\"" > /etc/rc.conf

### Huso horario

    # ln -sf /usr/share/zoneinfo/<Region>/<Ciudad> /etc/localtime

Ejemplo

    # ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime

Y actualizamos el reloj hardware

    # hwclock --systohc

### Locale

Descomentamos el idioma preferido en `/etc/locale.gen` (se puede hacer con cualquier editor, pero a mi me gusta complicarme la vida):

    # sed -i 's/\(#\)\(<lenguage>_<PAIS>\.UTF.*\)/\2/' /etc/locale.gen

Ejemplo:

    # sed -i 's/\(#\)\(es_ES\.UTF.*\)/\2/' /etc/locale.gen

Y lo generamos:

    # locale-gen

Definimos el lenguage local de nuestra máquina (`/etc/locale.conf`):

    # localectl set-locale LANG=es_ES.UTF-8

El mapeado de teclas de la consola virtual y de x11  (`/etc/vconsole.conf` y `/etc/X11/xorg.conf.d/00-keyboard.conf`):

    # localectl set-x11-keymap es

Si queremos un layout inglés con acentos al pulsar la tecla alt-gr hay que hacer un paso más:

    Descomentamos el lenguage
    # sed -i 's/\(#\)\(en_US\.UTF.*\)/\2/' /etc/locale.gen
    # locale-gen
    # localectl set-locale LANG=en_US.UTF-8
    # localectl set-keymap us altgr-intl
    # localectl set-x11-keymap us "" altgr-intl

### Nombre del equipo

Añadimos el nombre del equipo a `/etc/hostname`

    # echo bug > /etc/hostname

Y añadimos una entrada en `/etc/hosts` con el nombre

    >/etc/hosts
    127.0.0.1	localhost
    ::1		    localhost
    127.0.1.1	bug.localdomain bug

### Soporte TRIM para SSDs

    # systemctl enable fstrim.timer

### Añadir usuario

Cambiamos contraseña del root

    # passwd

Añadimos usuario

    # useradd -m -g users -G wheel,storage,power -s /bin/bash <nombre_usuario>

y le cambiamos la contraseña

    # passwd <nombre_usuario>

Lo añadimos como sudoer:

    # export EDITOR=nano
    # visudo

Descomentamos `%wheel ALL=(ALL) ALL` y añadimos la siguiente linea para que al ejecutar sudo nos pida la contraseña del root y no la del usuario

    Defaults rootpw

### Configurar mkinitcpio para que nos pille LVM

Añadimos lvm2 a las HOOKS de `mkinitcpio.conf`

    >/etc/mkinitcpio.conf
    ...
    HOOKS="...block lvm2 filesystems..."
    ...

Generar la entrada en `/boot`

    # cd /boot
    # mkinitcpio -p linux

### Bootloader

Nos aseguramos de que las efivars estan bien montadas

    # mount -t efivarfs efivarfs /sys/firmware/efi/efivars

Vamos a usar systemd-boot

    # bootctl install

y modificamos dos archivos

    >/boot/loader/loader.conf
    default  arch
    timeout  5

y la entrada linux

    >/boot/loader/entries/arch.conf
    title     Arch Linux
    linux     /vmlinuz-linux
    initrd    /initramfs-linux.img
    options   root=/dev/mapper/<nombre_grupo_volumenes>-<nombre_volumen_root> rw

Para procesadores intel con microarquitectura haswell o mayor (i3, i5 y i7 a partir de 4XX0) instalar `intel-ucode`:

    # pacman -S intel-ucode

Y modificamos el archivo `/boot/loader/entries/arch.conf` añadiendo la siguiente línea

    >/boot/loader/entries/arch.conf
    ...
    linux     /vmlinuz-linux
    initrd    /intel-ucode.img     # <--
    initrd    /initramfs-linux.img
    ...

Para tarjetas NVIDIA que tengan incompatibilidades con los drivers por defecto (Nouveau), podemos deshabilitar estos en los parámetros del kernel:

    >/boot/loader/entries/arch.conf
    ...
    options   root=... modprobe.blacklist=nouveau

### Finalizar Instalación básica

    # exit
    # umount -R /mnt
    # reboot

Si al reiniciar da el error `"cgroup: remount is not allowed"`

    # systemctl mask mkinitcpio-generate-shutdown-ramfs.service

# Post instalación del sistema

## Conexión a internet

Si no tenemos acceso a internet, volver a intentar [la conexión](#conexión-a-internet). Vamos a usar `NetworkManager` como gestor de red

    # pacman -S networkmanager network-manager-applet

Buscamos los dispositivos de cable e inalambrico con `ip link`

    # ip link
    ...
    2: enp2s0f0... # cableado
    3: wlp3s0bl... # inalambrico

Desactivamos  ambos

    # systemctl disable dhcpcd@enp2s0f0.service
    # systemctl disable netctl-auto@wlp3s0bl.service

Activamos el servicio de NetworkManager y reiniciamos

    # systemctl enable NetworkManager.service
    # reboot

## Interfaz gráfica

Instalamos X Window System

    # pacman -S xorg

Instalamos los drivers de intel

    # pacman -S xf86-video-intel

Instalamos el entorno de ventanas preferido

    # pacman -S xfce4

Y el Display Manager

    # pacman -S lxdm
    # systemctl enable lxdm.service
    # systemctl start lxdm.service

## Configuración del sonido

Para el sonido utilizaremos el driver ALSA y el servidor de sonido PulseAudio.

Instalamos PulseAudio:

    $ sudo pacman -S pulseaudio

Optativamente podemos instalar el pluggin de xfce4 (si hemos elegido ese WM)

    $ sudo pacman -S xfce4-pulseaudio-plugin

Instalamos ALSA utils:

    $ sudo pacman -S alsa-utils

Entramos en el mixer de alsa-utils con `alsamixer` y desmuteamos todos las columnas (mostrando todas con F5) pulsando `m` en cada columna que tenga `MM` en la parte de abajo (muteado) para cambiarlo a `00` con fondo amarillo.

# Instalación de paquetes útiles

Para instalar `aurman` (Arch Users Repository helper) hacemos lo siguiente:

Pimero, instalamos git:

	$ sudo pacman -S git

Después, clonamos el repositorio de aurman y construimos el paquete:

    $ cd /tmp/
    $ git clone https://aur.archlinux.org/aurman.git # Clonamos
    $ cat PKGBUILD # Echamos un vistazo a lo que se va a instalar
    $ makepkg -si  # Instalamos

## Paquetes útiles

Instalamos el administrador de archivos con su gestor de archivos

    $ aurman -S pcmanfm\      # Administrador de archivos
	            file-roller   # Gestor de archivos (zip, tar.gz, ...)

Y paquetes útiles:

	$ aurman -S tree arandr firefox unzip p7zip evince\
      	     	xfce4-screenshooter htop ristretto openssh

Paquetes de desarrollo:

    $ aurman -S atom emacs gedit

Paquetes de media:

    $ aurman -S playerclt vlc

Paquetes para bloqueo de pantalla (necesarios para el script `lock.sh` de `dotfiles/scripts/`):

    $ aurman -S imagemagick scrot i3lock

Paquetes de temas gtk:

    $ aurman -S arc-gtk-theme arc-icon-theme
