---
title: "The struggles of an Nvidia user on Linux"
description: "How to setup Nvidia drivers on Arch Linux"
date: 2025-09-04
author: Junovell
toc: true
math: false
isStarred: true
tags: ["guides", "linux", "nvidia"]
---

## Introduction

Are *you* struggling like me with an nvidia GPU on linux? are *you* also too broke to buy a new AMD gpu? lucky for you then, this guide is here to help.

> *This guide is only for arch users*

## Installation

For all decently modern GPUs download `nvidia-open nvidia-utils lib32-nvidia-utils libva-nvidia-driver`

If you want to run non-official linux kernels (zen / cachyOS / etc...) then also install `nvidia-open-dkms` and the relevant headers for the kernel (`linux-headers` for the default kernel)

Optionally you can also install `nvidia-settings` to monitor and configure your gpu, but it's not required and it lacks 99% of it's features on wayland.

Then Remove `kms` from the `HOOKS` array in `/etc/mkinitcpio.conf`

```diff
- HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
+ HOOKS=(base udev autodetect microcode modconf keyboard keymap consolefont block filesystems fsck)
```

Notes:

- No need to early load `nvidia_drm.modeset=1` for drivers above 560.35.03-5 (drm is required for xwayland)
- Only early load if you encounter issues `nvidia nvidia_modeset nvidia_uvm nvidia_drm` > `mkinitcpio.conf` `MODULES()`

If you're going to run a wayland based DE then you need to enable Nvidia's [Direct Rendering Manager](<https://download.nvidia.com/XFree86/Linux-x86_64/396.51/README/kms.html>).

`/etc/modprobe.d/nvidia.conf`

```txt
options nvidia_drm modeset=1
```

Rebuild your initramfs and reboot

```sh
$ sudo mkinitcpio -P
$ reboot
```

After rebooting verify that your GPU is working by running `nvidia-smi`

### Monitoring

To monitor your nvidia GPU you can use `nvidia-smi` with the `watch` command or install `nvtop` for a top-like UI.

## Config

### Performance

Enable some kernel parameters for better performance

`/etc/modprobe.d/nvidia.conf`

```txt
options nvidia NVreg_UsePageAttributeTable=1
```

This enables [Page attribute table](<https://en.wikipedia.org/wiki/Page_attribute_table>)

If your GPU and motherboard supports PCIe Gen 3 but the driver shows it running in Gen 2 mode then you can add `NVreg_EnablePCIeGen3=1` to the options above.

Other options that can help:

- `NVreg_EnableResizableBar` enables [Resizable BAR](<https://www.nvidia.com/en-us/geforce/news/geforce-rtx-30-series-resizable-bar-support/>) which requires GPU and motherboard support. Although this doesn't always result in a performance boost and it depends on your system and the apps you're running.

- `NVreg_DynamicPowerManagement=0x01` enables the GPU to transition to its lowest power state when no applications are utilizing the NVIDIA driver stack.

Rebuild **initramfs** to apply the changes

```sh
mkinitcpio -P
```

Verify the parameters are properly set after reboot by running the following

```sh
$ cat /proc/driver/nvidia/params
```

Sources:

- <https://wiki.archlinux.org/title/NVIDIA#Installation>
- <https://wiki.gentoo.org/wiki/NVIDIA/nvidia-drivers/en>
- <https://gist.github.com/denji/52b9b0980ef3dadde0ff3d3ccf74a2a6>
- <https://forums.developer.nvidia.com/t/understanding-nvidia-drm-modeset-1-nvidia-linux-driver-modesetting/204068/3>

### Nvidia No USB C

If you have an nvidia gpu with no type C usb you might get an error in journalctl `nvidia-gpu 0000:01:00.3: i2c timeout error e0000000`

Run `journalctl -b 0 --grep "nvidia-gpu"` to check

This is an easy fix, create a file `/etc/modprobe.d/blacklist_i2c-nvidia-gpu.conf`

```txt
blacklist i2c_nvidia_gpu
```

Source: <https://askubuntu.com/a/1289997>

## Environment Variables

Add these to your config file (either your shell or WM)

```sh
# to force GBM backend https://wiki.archlinux.org/title/Wayland#Requirements
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia

# hardware acceleration https://wiki.archlinux.org/title/Hardware_video_acceleration
LIBVA_DRIVER_NAME=nvidia
VDPAU_DRIVER=nvidia

# VA-API HW support, requires `libva-nvidia-driver`
NVD_BACKEND=direct

# EXPERIMENTAL

# Enables threaded optimizations in OpenGL.
# Set to 1 to allow multi-threaded improvements in rendering performance.
# !!! THIS BREAKS STEAM CLIENT ON HYPRLAND
# __GL_THREADED_OPTIMIZATIONS=1

# Disables Mesa’s error checking to improve performance.
# Set to 1 to disable error reporting (use with caution).
# MESA_NO_ERROR=1
```

Install `libva-utils` and run `vainfo` to verify that VA-API works.
Install `vdpauinfo` and run `vdpauinfo` to verify that VDPAU works.
Install `vulkan-tools` and run `vulkaninfo | grep VK_KHR_video_` to verify that vulkan video works.

Sources:

- <https://wiki.hypr.land/Nvidia/>
- <https://github.com/elFarto/nvidia-vaapi-driver/>
- <https://gist.github.com/denji/52b9b0980ef3dadde0ff3d3ccf74a2a6>
- <https://wiki.archlinux.org/title/Hardware_video_acceleration#Verification>

### Hyprland

```conf
opengl {
  nvidia_anti_flicker = true
}

cursor {
  # both required to enable hardware cursors on nvidia
  use_cpu_buffer = true
  no_hardware_cursors = 0
}
```
