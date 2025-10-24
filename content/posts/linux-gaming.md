---
title: "Gaming on linux in big 2025"
description: "a guide on how to setup linux for gaming"
date: 2025-01-31
author: Junovell
toc: true
math: false
isStarred: false
tags: ["guides", "linux", "gaming", "steam"]
---

## Introduction

2025 is the year of linux gaming (or at least that's when I fully switched to linux)

This guide is for arch linux and mainly focused on wayland users.

## Prerequisites

You must have configured your GPU's drivers before following this guide.
Check out [my NVIDIA guide](/guides/nvidia-on-linux.md) if that's what you're running.

## STEAM

Honestly this is all you need to run games

### Installation

Install `steam ttf-liberation vulkan-icd-loader lib32-vulkan-icd-loader`

`ttf-liberation` is optional for steam as a replacement for microsoft's arial font.

### Config

To improve Vulkan shader compilation process you can increase the number of threads used by creating a file in `~/.steam/steam/steam_dev.cfg`

```sh
unShaderBackgroundProcessingThreads 4
```

Or outright disable as it can hog a LOT of space ([up to 100GB in some cases](<https://github.com/ValveSoftware/Fossilize/issues/237>)), and [many other issues](<https://github.com/ValveSoftware/Fossilize/issues>).
steam > settings > Downloads > **Enable Shader Pre-caching**, and turn it off.

## Audio

Install `pipewire-pulse` and enable it, this is required to have audio in some games.

```sh
systemctl --user enable pipewire-pulse
```

## MangoHud

MangoHud is that fancy overlay youtubers use to brag about how good their PCs are (yeah i'm salty because i'm stuck with an old system).

### Installation

Install `mangohud` and `lib32-mangohud` (if you want to run x86 games).

If you're using steam you can enable it in the game's launch options like so:

```sh
mangohud %command%
```

### Config

This is a basic config that I use.

`~/.config/MangoHud/MangoHud.conf`

```ini
font_size=20
position=top-left
text_color=ffffff
background_alpha=0.3
background_color=020202
no_display
toggle_hud=F11

# CPU
cpu_stats
cpu_temp
cpu_color=007AFA

# GPU
gpu_stats
gpu_temp
gpu_color=00BD00
vram
vram_color=00801B

# System
ram
ram_color=B3000A

# Application
fps
arch
io_read
io_write
engine_version
gamemode
engine_color=B200B0

graphs=gpu_load,cpu_load,vram
```

## Gamemode

Gamemode can help improve your game's performance by altering some system config before running the game and reseting them after.

### Installation

Install `gamemode` and `lib32-gamemode`

Add your user to gamemode group

```sh
sudo usermod -aG gamemode your-name
```

Enable GameMode

```sh
systemctl --user enable gamemoded
```

### Config

If you're not gonna use the [gpu] modifications then create a file in `~/.config/gamemode.ini`.
If you will apply the [gpu] modifications then use`/etc/gamemode.ini` or `/usr/share/gamemode/gamemode.ini` (do not create the latter file manually).

Copy the default config from here and change it
<https://github.com/FeralInteractive/gamemode/blob/master/example/gamemode.ini>

My settings:

```toml
; everything default except renice
; Docs allow values 0–20, but values above 10 fail
renice=10
```

If you want to set a renice value higher than 10 then [check the Arch wiki](<https://wiki.archlinux.org/title/GameMode#Renicing_fails_when_set_to_less_than_-10>)

Note: the GPU config for NVIDIA currently (2025) doesn't work on wayland.

If you're using mangohud and want it to show gamemode status then you need to run it like this
`LD_PRELOAD="/usr/lib/libgamemode.so" gamemoderun mangohud %command%`

Sources:

- <https://wiki.archlinux.org/title/GameMode>
- <https://github.com/FeralInteractive/gamemode/issues/447#issuecomment-2016576665>

## Proton (And It's Flavors)

Proton is valve's JVM for windows games, it allows you to run stinky windows games on your glorious (Arch) Linux system.

> *Disclaimer: proton can't run all games, especially those that rely on anti-cheat software*

Proton has many versions but 4 standout

- **Proton x.x** this is the stable proton release, it doesn't guarantee it will run all games, but it will handle all games that were released before it
- **Proton Experimental** this is the beta branch of Proton, in my experience this is stable enough to replace proton stable and gets updated more frequently

Aside from the official proton provided by valve, there are some community forks that ship with more up to date libraries

- [Proton-GE](<https://github.com/GloriousEggroll/proton-ge-custom>) the most popular one from what I can see
- [Proton-TKG](<https://github.com/Frogging-Family/wine-tkg-git/>) my go to proton for wayland support

Proton-ge also has an AUR package `proton-ge-custom-bin` but it's not worth it imo as it takes up almost 3Gb of space.

### Config

You can configure proton by adding environment variables to your game's launch options in steam like this

```sh
PROTON_LOG=1 PROTON_USE_XALIA=0 %command%
```

A list of official proton variables can be found [on their github page](<https://github.com/ValveSoftware/Proton?tab=readme-ov-file#runtime-config-options>).

To enable certain options for all games using that proton version you can edit
`.steam/steam/steamapps/common/Proton X/user_settings.py` (`X` here is the proton version you're using)
There will be a a sample file with the same name, just make a copy of it and use it.

The file is a python dictionary (string, string) that will be passed as environment variables when you run a game with that proton version, so technically you can add any environment variable you want and it doesn't have to be related to proton.

Here's my config:

```py
user_settings = {
    # everything is default except the below

    "WINE_FULLSCREEN_FSR": "0",
    # this enables DLSS even for non RTX cards
    "PROTON_ENABLE_NVAPI": "1",
    # enable OBS game recording (obs-vkcapture)
    "OBS_VKCAPTURE": "1",
    # disable xalia, this is for steam deck on screen controls.
    "PROTON_USE_XALIA": "0",
    # log dir has to be full verbose path, no variables allowed
    # else it will be relative to the game's directory
    "PROTON_LOG_DIR": "proton-logs",

    # if you're using proton-ge or proton-tkg
    # enable wayland by default because it's way better than xwayland
    "PROTON_ENABLE_WAYLAND": "1",
}
```

## System Config

You can either apply these configs or install a custom kernel like [linux-tkg](<https://github.com/Frogging-Family/linux-tkg>) or [linux-zen](<https://github.com/zen-kernel/zen-kernel>).

> *Personally I prefer sticking to the normal kernel*

### Up Your Virtual Memory

`/etc/sysctl.d/80-gamecompatibility.conf`

```sh
vm.max_map_count = 2147483642
```

Apply changes without reboot

```sh
sysctl --system
```

Sources:

- <https://archlinux.org/news/increasing-the-default-vmmax_map_count-value/>
- <https://github.com/torvalds/linux/blob/v5.18/include/linux/mm.h#L178>

### Change Kernel Parameters For Response Time Consistency

These improved my FPS in elden ring nightreign from 20-35 to 30-40

`/etc/tmpfiles.d/consistent-response-time-for-gaming.conf`

```sh
#    Path                  Mode UID  GID  Age Argument
w /proc/sys/vm/compaction_proactiveness - - - - 0
w /proc/sys/vm/watermark_boost_factor - - - - 1
w /sys/kernel/mm/lru_gen/enabled - - - - 5
w /proc/sys/vm/zone_reclaim_mode - - - - 0
w /proc/sys/vm/page_lock_unfairness - - - - 1
w /proc/sys/kernel/sched_child_runs_first - - - - 0
w /proc/sys/kernel/sched_autogroup_enabled - - - - 1
w /proc/sys/kernel/sched_cfs_bandwidth_slice_us - - - - 3000
w /sys/kernel/debug/sched/base_slice_ns  - - - - 3000000
w /sys/kernel/debug/sched/migration_cost_ns - - - - 500000
w /sys/kernel/debug/sched/nr_migrate - - - - 8
```

If you use swap then add this, it lowers the priority of using swap

```sh
w /proc/sys/vm/swappiness - - - - 10
```

If you have 32Gb of ram or more and it's mostly unused then add this (will increase ram used)

```sh
w /proc/sys/vm/min_free_kbytes - - - - 1048576
w /proc/sys/vm/watermark_scale_factor - - - - 500
```

If your games don't use TCMalloc (source games use it, but not through proton)

```sh
w /sys/kernel/mm/transparent_hugepage/enabled - - - - madvise
w /sys/kernel/mm/transparent_hugepage/shmem_enabled - - - - advise
w /sys/kernel/mm/transparent_hugepage/defrag - - - - never
```

### Reduce Buffer Bloat On Storage Devices

Enable BFQ on all HDDs and SSDs, and disable the scheduler on all NVMEs.

`/etc/udev/rules.d/60-ioschedulers.rules`

```sh
# HDD
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"

# SSD
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

# NVMe SSD
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"
```

> *To target a single drive just set it's name instead of regex `KERNEL=="sda"`*

Force udev update without rebooting

```sh
sudo udevadm trigger
```

### Enable TRIM For SSDs

```sh
sudo systemctl enable fstrim.timer
```

### Set CPU To Performance Mode

> *This is not required if you're using GameMode*

On intel CPUs you can alter the power state (pstate) to make it prioritize performance.

First make sure your cpu has pstate and HWP enabled:

```sh
$ cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_driver
# prints 'intel_pstate' for each physical core you have
intel_pstate
intel_pstate
...

$ lscpu | grep hwp
# prints a long string with 'hwp' in red
Flags: ... hwp hwp_notify hwp_act_window hwp_epp vnmi ...
```

Next write a script to enable performance mode for all cores

```sh
#!/bin/bash
for CPUFREQ in /sys/devices/system/cpu/cpu*/cpufreq; do
    echo performance > "$CPUFREQ/scaling_governor" 2>/dev/null
    echo performance > "$CPUFREQ/energy_performance_preference" 2>/dev/null
done
```

The script has to be run as root to work!

Notes:

- You don't need to reboot the pc for this to take effect.
- You have to re-run the script every time you reboot the pc.
- You can't run this in the gamemode config because it requires root permissions.

### RISKY!!!

#### CPU Security Mitigations

According to some phoronix benchmarks [disabling CPU security mitigations can improve CPU performance by up to 20%](<https://www.phoronix.com/review/3-years-specmelt>). Now this is an **AWFUL** idea unless you're desperate for any extra performance you can get, and even then It's still an awful idea, but I still want to mention it for the sake of completeness.

Note that this only will benefit Intel 10th gen and below, and AMD 5000 series and below.

In your `/etc/default/grub`

```ini
GRUB_CMDLINE_LINUX_DEFAULT="...minitgations=off..."
```

Then re-create the grub config

```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Reboot your PC and run `lscpu` to verify that you're now Vulnerable :)

Sources:

- <https://wiki.archlinux.org/title/Security>
- <https://docs.kernel.org/admin-guide/hw-vuln/>
- <https://ditana.org/docs/the-installer/kernel_configuration/mitigation/>

### CachyOS

For more optimizations check out the [CachyOS wiki](<https://wiki.cachyos.org/>)
