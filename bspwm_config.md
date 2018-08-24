# Bspwm configuration guide

## Installation

Install `bspwm` (window manager) and `sxhkd` (keybindingsa daemon) packages:

```bash
$ sudo pacman -S bspwm sxhkd
```

And their dependencies:

```bash
$ sudo pacman -S libxcb xcb-util xcb-util-wm xcb-util-keysyms
```

## Configuration

Copy the config files to your config folder:

```bash
$ mkdir -p ~/.config/{bspwm,sxhkd}
$ mkdir -p ~/.config/{bspwm,sxhkd}
$ cp /usr/local/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/
$ cp /usr/local/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/
$ chmod u+x ~/.config/bspwm/bspwmrc
```

### Notification and Compositor daemons for bspwm

Install `dunst` and `compton` packages:

```bash
$ sudo pacman -S dunst compton
```

Then copy their config files to your user config folder:

```bash
$ mkdir -p ~/.config/{dunst,compton}
$ cp /usr/share/dunst/dunstrc ~/.config/dunst/
$ cp /etc/xdg/compton.conf ~/.config/compton/
```

### Post basic installation

Now launch daemons from `bspwm` script:

```bash
> ~/.config/bspwm/bspwmrc
...
################### Autostart ###################

# Sxhkd keybindings
pkill sxhkd
sxhkd &

# Dunst notificaitons
# pkill dunst
# dunst &

# Compton compositor
pkill compton
compton -b --config $XDG_CONFIG_HOME/compton/compton.conf &
```
