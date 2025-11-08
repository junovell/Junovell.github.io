---
title: "Ultimate Arch Linux Installation Guide"
description: "I use Arch btw"
date: 2025-09-02
author: Junovell
toc: true
math: false
isStarred: true
tags: ["guides", "linux"]
---

## Introduction

An ultimate guide for Arch linux, from a black screen to streaming bloons TD 6 to your friends on discord.

> You might notice the style of writing changes from section to section, that's because this guide took me a couple of weeks to finish

## Hardware

There aren't any hardware requirements to run linux, but you need to configure some stuff before starting

- Preferably enable UEFI mode in BIOS.
- Disable secure boot for an easy installation.
- If you have an NVME SSD then you need to disable RAID mode in the bios.

## Downloading Arch

Download the Arch ISO from [the official site](<https://archlinux.org/download/>), preferably with torrent then copy it to an USB using [Ventoy](<https://www.ventoy.net/en/download.html>) or [rufus](<https://rufus.ie/>).

> Make sure you select GPT for the partition type.

## Installation

Read the [official Installation guide](<https://wiki.archlinux.org/title/Installation_guide>) on the Arch wiki until you reach the [**Installation** section](<https://wiki.archlinux.org/title/Installation_guide#Installation>) then you can continue with my guide.

```sh
pacstrap -K /mnt base base-devel linux linux-firmware git networkmanager nano efibootmgr sudo
```

You can also add:

- `amd-ucode` if you have an AMD cpu
- `intel-ucode` if you have an Intel cpu
- `fish` or `zsh` if you want a different shell
- `os-prober` for dual-boot if you have a Windows installation on the system

Then chroot into the system

```sh
arch-chroot /mnt
```

And enable the network manager

```sh
systemctl enable NetworkManager
```

### Bootloader

On Linux you have [many options for booting your system](<https://wiki.archlinux.org/title/Arch_boot_process#Feature_comparison>), but for this guide I'll only go through **GRUB** and **EFI Stub**.

- **GRUB** is a popular bootloader that provides a UI at boot time, allowing you to select from multiple OSes. It supports theming, fallback configurations, and advanced options like chainloading other bootloaders. GRUB is configured through a configuration file (`grub.cfg`), which can be generated or modified manually.
- **EFI Stub** uses your Linux kernel as a direct EFI executable. When using EFI stub, the kernel is loaded directly by the UEFI firmware, making the boot process faster. However, it lacks a graphical user interface or boot menu, and switching between different operating systems requires configuring the boot options directly in the BIOS.

> If you're still new just use **GRUB** and enjoy theming it, if you want your PC to boot as fast as possible then use **EFI Stub**

Regardless of which loader you use you need to install `efibootmgr`.

#### GRUB

Although there are other boot managers, grub has the most features and can be themed

Install `grub`.

```sh
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
$ nano /etc/default/grub # enable OS_PROBER and verbose logging
$ grub-mkconfig -o /boot/grub/grub.cfg
```

> If you have an MSI motherboad then you need to add `--removable` to the `grub-install` command.

**Then reboot.**
Source: <https://wiki.archlinux.org/title/GRUB#Installation>

#### EFI Stub

To create an EFI entry for a kernel:

```sh
efibootmgr --create --disk /dev/sdX --part <Y> --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=UUID=<drive-uuid> rw loglevel=3 verbose initrd=\initramfs-linux.img'
```

- `/dev/sdX` is the drive containing your boot partition
- `<Y>` is the boot partition's number
- `<drive-uuid>` is the UUID of your root partition, you can find it with `lsblk -o "NAME,UUID,MOUNTPOINTS"`

an example with my system:

```sh
sudo efibootmgr --create --disk /dev/sda --part 1 --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=e1f5c537-cb17-4d24-98c9-00b5085856b4 rw loglevel=3 verbose initrd=\initramfs-linux.img'
```

**Then reboot.**

Source: <https://wiki.archlinux.org/title/EFI_boot_stub#Using_UEFI_directly>

#### Deleting Bootloader

If you have multiple bootloaders configured and want to delete the ones you don't need anymore you can delete them.

First list your bootloaders

```sh
$ efibootmgr
# Boot0000* Arch Linux Stub HD(1,GPT,e1f5c537-cb17-4d24-98c9-00b5085856b4,0x800,0x200000)/\vmlinuz-linux72006f006f0074003d005500550049004400...
# Boot0001* GRUB HD(1,GPT,e1f5c537-cb17-4d24-98c9-00b5085856b4,0x800,0x200000)/\EFI\GRUB\grubx64.efi
```

Then delete the bootloader using it's id, for example I'll delete the GRUB entry:

> while the previous command lists GRUB as `Boot0001`, when deleting you only need to write the number `1`.

```sh
sudo efibootmgr --delete-bootnum --bootnum "1"
```

### Account

Set up your PC's Hostname, this is a [label that is assigned to a device connected to a network that is used to identify the device.](<https://en.wikipedia.org/wiki/Hostname>)

```sh
echo pc-name > /etc/hostname
```

#### Create a user account

I prefer [fish shell](<https://fishshell.com/>) because it's *fish* and *funny*, so I’ll create my user with that as the *default shell*.

> Fish shell also has some [other features](<https://fishshell.com/docs/current/interactive.html>) but they're not as important.

I prefer using Fish shell because, well, it's hooked me with its user-friendly features—and it's just fin-tastic.

> Sure, Fish shell has a bunch of other cool features, but let's be honest... it's mostly about being the catch of the day.

```sh
$ passwd
$ useradd -mG wheel -s /bin/fish melty
$ passwd melty
$ nano /etc/sudoers
```

> If you prefer another shell (like bash or [zsh](<https://www.zsh.org/>)), then replace `/bin/fish` with it's path, or omit the `-s` flag entirely to default to **Bash**.

In the `/etc/sudoers` file just scroll down and uncomment the rule for wheel, this will give all users in the wheel group access to sudo.

```diff
- # %wheel ALL=(ALL:ALL) ALL
+ %wheel ALL=(ALL:ALL) ALL
```

If you want only your user to have sudo access then you can re-write the live but with your name at the start

```txt
melty ALL=(ALL:ALL) ALL
```

### Install yay

**yay** is an AUR helper, there's also **paru** but **yay** sounds better

```sh
$ git clone https://aur.archlinux.org/yay.git
$ cd yay && makepkg -si
$ yay -Y --gendb # init
$ yay -Y --devel --save # always check for -git apps
$ yay -S less # less is an optional package for yay
$ cd .. && rm -rf yay
```

### configure yay and pacman

```sh
# edit as you wish
nano ~/.config/yay/config.json
```

```sh
# enable multilib and colors
sudo nano /etc/pacman.conf
```

```diff
# Misc options
#UseSyslog
- #Color
+ Color
#NoProgressBar
CheckSpace
#VerbosePkgLists
ParallelDownloads = 4

...

- #[multilib]
- #Include = /etc/pacman.d/mirrorlist
+ [multilib]
+ Include = /etc/pacman.d/mirrorlist
```

```sh
# initialize pacman
$ sudo pacman-key --init && sudo pacman-key --populate
$ yay # update the system
```

### Add a GPG server

This is required to verify some packages on the AUR

```sh
$ mkdir ~/.gnupg
$ echo "keyserver hkps://keyserver.ubuntu.com" >> ~/.gnupg/gpg.conf
```

### Install a UI

For the login manager i'll use **SDDM** because it's customizable, and for the system UI I'll go with **Hyprland** because from my trials it's the best wayland window manager for nvidia users, also the config is super easy

```sh
$ yay -S sddm # install `weston` if you want to launch sddm in wayland mode
$ yay --needed -S hyprland hyprpaper hypridle xdg-desktop-portal-hyprland hyprpolkitagent
$ sudo systemctl enable sddm
```

Some other nice alternatives for hyprland:

- [Sway](<https://github.com/swaywm/sway>)
- [River](<https://codeberg.org/river/river>)
- [Maomaowm](<https://github.com/DreamMaoMao/maomaowm>)

Basic apps to pair with hyprland

```sh
yay -S waybar rofi-wayland foot wl-clip-persist
```

> *Pick pipewire-jack when inatalling waybar*

Edit `~/.config/hypr/hyprland.conf` to use the above packages.

I like to organize my config in seperate files, so i create a `sources` folder in `hypr`

```sh
mkdir ~/.config/hypr/sources
```

In it I'll make `keybinds.conf`, `startup.conf`, `envs.conf` and `window-rules.conf`
here are some config you need

`startup.conf`

```sh
# https://wiki.hypr.land/Hypr-Ecosystem/xdg-desktop-portal-hyprland/#share-picker-doesnt-use-the-system-theme
exec-once = dbus-update-activation-environment --systemd --all && systemctl --user import-environment QT_QPA_PLATFORMTHEME WAYLAND_DISPLAY XDG_CURRENT_DESKTOP

# https://gist.github.com/brunoanc/2dea6ddf6974ba4e5d26c3139ffb7580#editing-the-configuration-file
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP

# https://wiki.hypr.land/Hypr-Ecosystem/hyprpolkitagent/#usage
exec-once = systemctl --user start hyprpolkitagent

# https://wiki.hypr.land/Hypr-Ecosystem/hypridle/#configuration
exec-once = hypridle

exec-once = dunst & hyprpaper & waybar & kdeconnect-indicator

# https://github.com/Linus789/wl-clip-persist#usage
exec-once = wl-clip-persist --clipboard regular
```

Then import them in your hypr config
`~/.config/hypr/hyprland.conf`

```sh
source = sources/envs.conf
source = sources/nvidia.conf
source = sources/startup.conf
source = sources/keybinds.conf
source = sources/window-rules.conf
```

As of June 19 2025 **hyprpolkitagent** will fail to run on boot

- Reported in: <https://github.com/hyprwm/hyprpolkitagent/issues/29>
- Fixed in: <https://github.com/hyprwm/hyprpolkitagent/pull/30>

### Install GPU driver

Read the [Nvidia guide](/guides/nvidia-on-linux)

### Install apps

Must have apps

```sh
$ yay -S dolphin vesktop librewolf-bin zed dunst
# i chose ttf-liberation as i need it for steam later
```

- `dolphin`: File explorer.
- `vesktop`: Discord, but cuter.
- `librewolf-bin`: Tracking stripped fork of firefox in binary form, remove the `-bin` if you believe your CPU can compile a browser.
- `zed`: VS Code alternative, but if you want vscode then use `vscodium`.
- `dunst`: Notification daemon.

After installing `dolphin` if it can't open any apps by default try running this command

```sh
sudo ln -s /etc/xdg/menus/plasma-applications.menu /etc/xdg/menus/applications.menu
```

<https://discuss.kde.org/t/dolphin-doesnt-show-a-single-app-in-the-open-with-menu/14799/12>

Candy

```sh
$ yay -S kdeconnect kio-admin ffmpegthumbs sshfs yt-dlp-git qt6ct fastfetch
```

- `qt6ct`: Theming for QT6 apps
- `sshfs`: Browsing phone storage from dolphin when connected with kdeconnect.
- `kio-admin`: Managing files in dolphin as root.
- `fastfetch`: Continuation of `neofetch`.
- `kdeconnect`: Application to link your phone with your pc and share files.
- `yt-dlp-git`: CLI tool to download videos from youtube, twitter etc... (mpv can also use it to play online videos).

### Install audio

```sh
yay --needed -S pipewire lib32-pipewire wireplumber pwvucontrol pipewire-alsa pipewire-jack lib32-pipewire-jack
```

If you have a laptop then also install `sof-firmware` if that doesn't work then also `alsa-firmware`

Enable wireplumber and pipewire for the user

```sh
$ systemctl --user enable wireplumber
$ systemctl --user enable pipewire
```

Or globally with

```sh
$ systemctl --global enable wireplumber
$ systemctl --global enable pipewire
```

#### Voice noise suppression

For a system level solution for noise suppression

```sh
$ yay -S noise-suppression-for-voice pipewire-pulse
$ systemctl --user enable pipewire-pulse
```

And follow [the project's guide](<https://github.com/werman/noise-suppression-for-voice#pipewire>)

*For Arch linux the plugin's path is `/usr/lib/ladspa/librnnoise_ladspa.so`*

### Install media players

I only need a video player and an Image viewer

```sh
yay -S mpv nomacs-git
```

using the git version of nomacs because the version available in the arch repo is old and doesn't work with the latest openssl package

### Install fonts

For languages:

```sh
yay -S noto-fonts noto-fonts-cjk noto-fonts-extra
```

For emojis (pick at least 1):

- `ttf-twemoji-color` (i use this)
- `noto-fonts-emoji` (i use this)
- `ttf-font-awesome`
- `otf-openmoji`

If you installed `ttf-twemoji-color` then run this

```sh
sudo ln -sf /usr/share/fontconfig/conf.avail/46-ttf-twemoji-color.conf /etc/fonts/conf.d/46-ttf-twemoji-color.conf
```

Set up font fallback, for my case this fixes Noto fonts using the Urdu font for arabic and prioritizes twitter emojis

`~/.config/fontconfig/fonts.conf`

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
    <alias>
        <family>sans-serif</family>
        <prefer>
            <family>Noto Sans</family>
            <!-- find the others here https://fonts.google.com/noto/fonts?query=arabic -->
            <family>Noto Sans Arabic</family>
            <!-- chinese and stuff -->
            <family>Noto Sans CJK SC</family>
            <!-- ttf-twemoji-color -->
            <family>Twitter Color Emoji</family>
            <!-- fallback to noto-fonts-emoji -->
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <alias>
        <family>serif</family>
        <prefer>
            <family>Noto Serif</family>
            <family>Noto Naskh Arabic</family>
            <family>Twitter Color Emoji</family>
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <alias>
        <family>monospace</family>
        <prefer>
            <family>Noto Sans Mono</family>
            <family>Noto Kufi Arabic</family>
            <family>Twitter Color Emoji</family>
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>
</fontconfig>
```

Enable some presets to improve readability on 1080p screens and lower

```sh
$ mkdir -p ~/.config/fontconfig/conf.d
$ ln -s /usr/share/fontconfig/conf.avail/70-no-bitmaps.conf ~/.config/fontconfig/conf.d/
$ ln -s /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf ~/.config/fontconfig/conf.d/
$ ln -s /usr/share/fontconfig/conf.avail/10-hinting-slight.conf ~/.config/fontconfig/conf.d/
$ ln -s /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf ~/.config/fontconfig/conf.d/
```

### Add drives

Time to add your extra storage drives

If you want to mount an NTFS drive then install `ntfs-3g`

First get your drive's ID by running

```sh
$ lsblk -o "NAME,FSTYPE,UUID,SIZE,MOUNTPOINTS"
```

Then add it to fstab

`/etc/fstab`

```sh
# for ext4 drives
UUID=XXXXXXX /mnt/linux ext4 rw,relatime 0 1
# for NTFS drives
UUID=XXXXXXX  /mnt/windows ntfs uid=1000,gid=1000,auto,rw,user,exec,windows_names,umask=000 0 0
```

after adding an EXT2/EXT3/EXT4/XFS/BTRFS (or any other POSIX-compliant format) drive, and you want it to be writable by your user you need to run the following:

```sh
sudo chown -R $USER:$USER /mnt/linux # replace with your drive's mountpoint obv
```

Notes:

- if your drive doesn't have any system files then disable `fsck` by putting a 0 at the end instead of 1
- `/mnt/linux` and `/mnt/windows` are placeholders and can be anywhere on your system, just make sure you create the folder manually
- Optionally you can add `big_writes` to drives that you'll mostly be reading and writing into in big chunks (game storage for example)
- Optionally you can add `noatime` to skip modifying the files' access time on the drive which will improve read performance slightly.

**Reboot to apply changes**

### Gaming

This part is too long so I moved it into it's own post [Gaming on Linux](</guides/linux-gaming.md>)

## OBS

Install `obs-studio`

- `aur/wlrobs`: Native wlroot recording.
- `v4l2loopback-dkms`: Virtual camera support.
- `obs-vkcapture` / `lib32-obs-vkcapture`: Vulkan and OpenGL apps capture (games),
You also need to run the app with `OBS_VKCAPTURE=1`

### Hyprland Global keybinds

Hyprland supports global keybinds, but it's a bit weird, first setup your keybinds in OBS then in your hypr config add

```sh
bind = SUPER, F10, pass, class:^(com\.obsproject\.Studio)$
bind = , F11, pass, class:^(com\.obsproject\.Studio)$
```

Notice on the second line there's an empty space then a comma, that's because Hyprland expects 2 keys, so if you want to have a single key passed you need to leave the first or second one empty

## Development

### Git

Install `git` and `openssh`

`openssh` is required to use services that provide git hosting (github, gitlab, codeberg, etc...)

### Podman

Read the [podman guide](</guides/podman>)

## Configuration

### SDDM

```sh
$ sudo mkdir /etc/sddm.conf.d
$ sudo cp /usr/lib/sddm/sddm.conf.d/default.conf /etc/sddm.conf.d/general.conf
$ sudo nano /etc/sddm.conf.d/general.conf # have fun
```

Enable Numlock on SDDM start, only works for X11 session

```conf
[General]
Numlock=on
```

Change SDDM keyboard layout, this also only works for X11 session

```sh
sudo localectl set-x11-keymap ar
```

### Hyprland

I won't post my entire config, but here are some snippets

#### environment variables

Set GTK to fallback to X11 if wayland isn't possible

```sh
env = GDK_BACKEND,wayland,x11,*
```

Set QT to fallback to X11 if wayland isn't possible, and set the theme manager to `qt6ct`

You also need to install `qt6-wayland` or `qt5-wayland`, you can install both, but i'd recommend only installing the QT6 package for now unless you need the other one

```sh
env = QT_QPA_PLATFORM,wayland;xcb
env = QT_QPA_PLATFORMTHEME,qt6ct # can fix some missing functionality in QT apps if qt5ct is installed
```

To run [clutter](<https://archlinux.org/packages/?name=clutter>) apps with wayland

```sh
env = CLUTTER_BACKEND,wayland
```

For SDL3 wayland is used by default but to enable it on SDL2

```sh
env = SDL_VIDEODRIVER,wayland,x11
```

It's recommened to always have X11 as a fallback, you can also install `libdecor` to enable window decoration

GLFW supports wayland with

```sh
env = XDG_SESSION_TYPE,wayland
```

For electron apps using electron 28 and above

```sh
env = ELECTRON_OZONE_PLATFORM_HINT,wayland # or auto
```

For all electron apps you need a seperate config file

`~/.config/electron-flags.conf`

```txt
--ozone-platform-hint=auto
--enable-features=WaylandWindowDecorations
# uncomment to enable apps to sharescreen with pipewire
# --enable-features=WebRTCPipeWireCapturer
```

Or `~/.config/electronXX-flags.conf` where *XX* is Electron version to target a specific electron version

Note: These configuration files only work for the Electron packages in the official repositories and packages that use them. They do not work for packages that bundle their own build of Electron such as `slack-desktop`. Sometimes alternatives exist such as `slack-electron`.

#### latency

Enabling tearing may improve responsiveness of games

```sh
general {
    allow_tearing = true
}

windowrule = immediate, class:^(cs2)$
```

Source: <https://wiki.hypr.land/Configuring/Tearing/>

#### Zooming

Make a keybind that calls a script

```sh
# zoom in and out with the +/- numpad keys
bind = $mainMod, code:86, exec, ~/.config/hypr/scripts/zoomin.sh
bind = $mainMod, code:82, exec, ~/.config/hypr/scripts/zoomout.sh
```

You need to install `bc` for the math, also note that the scripts need to be bash because fish doesn't support `<<<`
the scripts are

`zoomin.sh`

```sh
#!/bin/bash

# current zoom
CURZOOM=$(hyprctl getoption cursor:zoom_factor | grep float)
CURZOOM=${CURZOOM:7:8} # https://stackoverflow.com/a/428580

# next zoom
NEXTZOOM=$(bc -l <<<"${CURZOOM}+0.5") # https://stackoverflow.com/a/34286545

hyprctl keyword cursor:zoom_factor $NEXTZOOM
```

`zoomout.sh`

```sh
#!/bin/bash

# current zoom
CURSZOOM=$(hyprctl getoption cursor:zoom_factor | grep float)
CURSZOOM=${CURSZOOM:7:8} # https://stackoverflow.com/a/428580

# next zoom
NEXTZOOM=$(bc -l <<<"${CURSZOOM}-0.5")

if [ $(bc -l <<<"${NEXTZOOM} < 1") == 1 ]; then
    NEXTZOOM=(1)
fi

hyprctl keyword cursor:zoom_factor $NEXTZOOM
```

#### Hypridle

I simply want my screen to turn off after 5 min to save energy, i don't want to lock it

`~/.config/hypr/xdph.conf`

```sh
general {
    after_sleep_cmd = hyprctl dispatch dpms on  # to avoid having to press a key twice to turn on the display.
}

# turn off keyboard backlight, comment out this section if you dont have a keyboard backlight.
# listener {
#     timeout = 150                                          # 2.5min.
#     on-timeout = brightnessctl -sd rgb:kbd_backlight set 0 # turn off keyboard backlight.
#     on-resume = brightnessctl -rd rgb:kbd_backlight        # turn on keyboard backlight.
# }

listener {
    timeout = 300                                 # 5min
    on-timeout = hyprctl dispatch dpms off        # screen off when timeout has passed
    on-resume = hyprctl dispatch dpms on          # screen on when activity is detected after timeout has fired.
}
```

Source: <https://wiki.hypr.land/Hypr-Ecosystem/hypridle/>

#### Hyprpaper

*A nice wallpaper~*
I only use 1 wallpaper on both my monitors so this is enough for me

```sh
preload = ~/Pictures/wallpaper/mono1.png
wallpaper = , ~/Pictures/wallpaper/mono1.png
```

If you want to specify the a wallapaper for each monitor then you need to get your monitors name

```sh
hyprctl monitors
```

Example output:

```txt
Monitor HDMI-A-1 (ID 1):
  1920x1080@60.00000 at 0x0
  description: Samsung Electric Company S24F350 H4ZM400172
  make: Samsung Electric Company
  model: S24F350
  serial: H4ZM400172
  active workspace: 4 (4)
  special workspace: 0 ()
  reserved: 0 0 0 40
  scale: 1.00
  transform: 0
  focused: yes
  dpmsStatus: 1
  vrr: false
  solitary: 0
  activelyTearing: false
  directScanoutTo: 0
  disabled: false
  currentFormat: XRGB8888
  mirrorOf: none
  availableModes: 1920x1080@60.00Hz 1920x1080@59.94Hz 1920x1080@50.00Hz...
```

My monitor's name is `HDMI-A-1`, so the config would be

```sh
preload = ~/Pictures/wallpaper/mono1.png
wallpaper = HDMI-A-1, ~/Pictures/wallpaper/mono1.png
preload = ~/Pictures/wallpaper/mono2.png
wallpaper = HDMI-A-2, ~/Pictures/wallpaper/mono2.png
```

Source: <https://wiki.hypr.land/Hypr-Ecosystem/hyprpaper/>

### XDG Desktop Portal

#### Configure XDPH

XDPH stands for "Xdg Desktop Portal Hyprland"

`~/.config/hypr/xdph.conf`

```sh
screencopy {
    max_fps = 60
    allow_token_by_default = true
}
```

#### Use a Portal with a file picker

XDPH doesn't come a with file picker so you'll need to install another portal that implements that functionality.

You can find a list of available portals on arch and their supported interfaces on [the wiki](<https://wiki.archlinux.org/title/XDG_Desktop_Portal#List_of_backends_and_interfaces>).

For this guide i'll show you how to use kde or gnome's implementation, although i'd recommend KDE because it has the most support

First download the portal `xdg-desktop-portal-kde` / `xdg-desktop-portal-gtk` (GTK 3) or `xdg-desktop-portal-gnome` (GTK 4)

Then create `~/.config/xdg-desktop-portal/hyprland-portals.conf` and paste this

```toml
[preferred]
default = hyprland;kde
org.freedesktop.impl.portal.FileChooser = kde
```

Change `kde` with the name in your package, so if you downloaded `xdg-desktop-portal-dde` (the deepin portal) then write `dde`.

Source: <https://wiki.hypr.land/Hypr-Ecosystem/xdg-desktop-portal-hyprland/>

### Pacman & yay

#### Optimize pacman compilation

`/etc/makepkg.conf`

```diff
#-- Compiler and Linker Flags
#CPPFLAGS=""
- CFLAGS="-march=x86-64 -mtune=generic -O2 -pipe -fno-plt -fexceptions \
+ CFLAGS="-march=native -O2 -pipe -fno-plt -fexceptions \
        -Wp,-D_FORTIFY_SOURCE=3 -Wformat -Werror=format-security \
        -fstack-clash-protection -fcf-protection \
        -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer"
```

```diff
#-- Make Flags: change this for DistCC/SMP systems
- #MAKEFLAGS="-j2"
+ MAKEFLAGS="--jobs=$(nproc)"
```

```diff
- OPTIONS=(strip docs !libtool !staticlibs emptydirs zipman purge debug lto)
+ OPTIONS=(strip docs !libtool !staticlibs emptydirs zipman purge !debug lto)
```

```diff
- COMPRESSZST=(zstd -c -T0 -)
+ COMPRESSZST=(zstd -c -T0 --auto-threads=logical --ultra -22 -)
```

Then rebuild your manually installed packages with

```sh
yay -Qqt | yay -Syu --rebuild --answerclean None -
```

#### Clean old / unused packages

I use a fish function for this

```fish
function yay_clean
    echo "cleaning yay cache..."
    yay -Sc --noconfirm # remove old packages from cache
    # yay -Scc --noconfirm # remove ALL packages from cache
    yay -Yc --noconfirm # remove unneeded dependencies
end
```

#### Update mirrors

You can use `rate-mirrors` to update pacman's mirrors with the latest and fastest mirrors for your internet.

I have it setup as a fish function that updates mirrors and does a system wide update

```fish
function yay_update_all
    set TMPFILE (mktemp)

    if not sudo true
        echo need sudo permission to run...
        exit
    end

    rate-mirrors --save=$TMPFILE arch --max-delay=21600
    sudo mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-backup
    sudo mv $TMPFILE /etc/pacman.d/mirrorlist
    yay_clean # from previous step
    yay
end
```

Source: <https://github.com/westandskif/rate-mirrors>

#### Use ALHP

> Buildbot for Archlinux based repos with different x86-64 feature levels, -O3 and LTO.

1. Verify your system's feature level support

```sh
/lib/ld-linux-x86-64.so.2 --help
```

Example output for `x86-64-v3` support

```txt
Subdirectories of glibc-hwcaps directories, in priority order:
  x86-64-v4
  x86-64-v3 (supported, searched)
  x86-64-v2 (supported, searched)
```

2. Install keyrings and mirrorlist

```sh
yay -S alhp-keyring alhp-mirrorlist
```

3. Add the mirrorlists to your `/etc/pacman.conf`

```toml
[core-x86-64-v3]
Include = /etc/pacman.d/alhp-mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

[extra-x86-64-v3]
Include = /etc/pacman.d/alhp-mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

# if you need [multilib] support
[multilib-x86-64-v3]
Include = /etc/pacman.d/alhp-mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist
```

Replace the `v3` at the end with the highest feature level your cpu supports then update your system.

Note that the repos must be in that order so that **v3** is prioritized over

Source: <https://somegit.dev/ALHP/ALHP.GO>

#### Change cache to /tmp

if you don't need pacman's cache at all you can change it's directory to `/tmp/pacman` that way the cache is always deleted on shutdown

## Misc

### Quality of life scripts

First how to manage custom scripts in linux

- **fish**

Create the script under `~/.config/fish/functions/my_script.fish` and they'll be available for that user when using fish

- **bash**

Bash doesn't have a scripts folder so you'll need to make one yourself, I recommend `~/.scripts`.

Then add them to your bash config file

`~/.bashrc`

```sh
alias my_script='~/.scripts/my-script.sh'
```

Like fish, the aliases will only be available for that user when using bash, but unlike fish if you create the scripts in a root folder, let's say `/var/scripts`, then all users can alias them (but you'll need root to run them)

- **Hyprland**

Hyprland also doesn't have a scripts folder, so We'll use `~/.scripts` again

`~/.config/hypr/hyprland.conf`

```toml
bind = $mainMod, SPACE, exec, ~/.scripts/my-script.sh
```

#### Screenshots

Either install `spectacle` if you want an out of the box great experience, or `grim`, `slurp` and `swappy` then a make a script that you can call with hyprland binds

```sh
#!/bin/bash
grim -g "$(slurp)" - | swappy -f -
```

#### Math

Install `bc` to do calculations in the terminal, make a bash file and call it

> fish

```fish
#!/bin/fish
function calc
    echo $argv | bc
end
```

> bash

```bash
#!/bin/bash
res=$(bc -l <<< "$@");
echo $res;
```

One minor issue with the fish script is that it doesn't show denominators, for example `3 / 2` will only output `1` in fish

### System monitoring

It's useful to know the state of your system, for that you can use the following apps

- `htop` / `bottom`: general system info
- `nethogs`: monitoring network traffic
- `nvtop`: GPU info

### Archiving tools

Install either `ark` (by kde) or `xarchiver` (by gnome), then install

- `arj`: ARJ format support
- `lrzip`: LRZ format support
- `lzop`: LZO format support
- `7zip`: 7Z format support
- `unarchiver`: RAR format support
- `unrar`: RAR decompression support

### Terminal download manager

**Curl** is nice, but `aria2` is something else, it has support for direct and torrent download and can deal with HTTP redirects without any extra arguments.

### Journald

to save a bit more of space you can configure max size for Journald logs in
`/etc/systemd/journald.conf`

```sh
Compress=yes
SystemMaxUse=100M # maximum size in MB for saved logs
MaxFileSec=1month
```

you can find saved logs in `/var/log/journal/<id>`
