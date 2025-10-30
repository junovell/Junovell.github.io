---
title: "You're telling me there's a MAN in that POD???"
description: "How to install and use Podman on Arch"
date: 2025-09-16
author: Junovell
toc: true
math: false
isStarred: false
tags: ["guides", "linux", "podman"]
---

## Installation

Install `podman`, an open source replacement for docker, although it does lack some features.

## Enable Registries

`/etc/containers/registries.conf`

```diff
# # An array of host[:port] registries to try when pulling an unqualified image, in order.
- # unqualified-search-registries = ["example.com"]
+ unqualified-search-registries = ["docker.io"]
```

Sources:

- <https://podman.io/docs/installation#registriesconf>
- <https://kresna.dev/solving-podman-error-short-name-did-not-resolve-to-an-alias-and-no-unqualified-search-registries/>

## Pass USB to container

Just launch it with `--privileged` and check with `lsusb` or under `/dev/bus/usb/*` you should find the same devices as on the host

## Listen to audio

If you want to listen to audio from within your container then you need to pass the following argument

```sh
-v $XDG_RUNTIME_DIR/pulse/native:$XDG_RUNTIME_DIR/pulse/native
```

And install `pulseaudio` (if you're running debian)

## Run GUI apps inside podman

You can run GUI apps inside containers very easily actually, just run your container with

### Wayland (root not required)

```sh
podman run --env DISPLAY --env WAYLAND_DISPLAY --env XDG_RUNTIME_DIR -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY your_image
```

or

```sh
podman run --network host --env DISPLAY --env WAYLAND_DISPLAY --env XDG_RUNTIME_DIR -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:/tmp/wayland.sock:z your_image
```

Another solution is to use `--security-opt label=type:container_runtime_t`, but it's not secure because it weakens SELinux confinement and gives the container broader access to the host system.

```sh
podman run --network host --env DISPLAY --security-opt label=type:container_runtime_t your_image your_gui_app
```

source: <https://discussion.fedoraproject.org/t/how-can-i-create-a-container-with-podman-that-runs-graphical-application-in-isolation-from-the-file-system/73520/5>

a more secure solution is doing

```sh
waypipe --dir=/tmp/waypipe-docker --pulse docker run --rm -it \
  -e WAYLAND_DISPLAY=wayland-0 \
  -v /tmp/waypipe-docker:/tmp/waypipe-docker \
  -v /dev/snd:/dev/snd \
  firefox-wayland
```

source: <https://gitlab.freedesktop.org/mstoeckl/waypipe>

### X11

this might work, but i didn't test it

```sh
podman run --network host --env DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:ro,z my_image firefox
```

or the unsafe version

```sh
podman run -v $XAUTHORITY:$XAUTHORITY:ro -v /tmp/.X11-unix:/tmp/.X11-unix:ro --userns keep-id --env "DISPLAY" --security-opt label=type:container_runtime_t your_image your_gui_app
```

source: <https://discussion.fedoraproject.org/t/how-to-run-a-gui-app-in-a-podman-container/72970/8>
