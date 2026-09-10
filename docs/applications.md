# Applications & App Hub

Because host `pacman` is removed to guarantee display-stack immutability, extra
applications and development dependencies are installed **inside an Arch Linux
Distrobox container** instead of on the host.

Users never need to touch a terminal to manage, launch, or install applications.

## Horizon App Hub (`horizon-app-drawer`)

A graphical application launcher, installer, and container manager
(`/usr/local/bin/horizon-app-drawer`, a GTK 3 app). From it you can:

- **Launch anything.** Host desktop software and isolated Arch Distrobox apps launch
  from the same interface. Container apps appear with an `[Arch App]` badge in the
  App Hub and in the normal GNOME application menu.
- **Search & install.** The "Get Applications" tab searches for packages, provisions
  them into the container, and exports their desktop shortcuts in the background.
- **Snapshots.** Trigger an on-demand system backup or open Timeshift directly from
  the UI.

## Terminal access (optional)

Advanced users can run:

```bash
horizon-apps
```

This initializes (on first run) and enters the dedicated **`arch-apps`** Distrobox
container, which has full `pacman` and AUR tooling. If a bundled offline rootfs is
present at `/var/lib/distrobox/archlinux-rootfs.tar.zst` it is imported; otherwise
the container is pulled from `docker.io/library/archlinux:latest`.

`horizon-apps` also ensures rootless `podman` has `/etc/subuid` and `/etc/subgid`
mappings for your user.

!!! tip
    Anything you install inside the container and export shows up automatically in
    the desktop menu — there is usually no reason to keep a terminal open.

## Flatpak

Flatpak is pre-configured for standalone desktop apps and is a good choice for
software you want isolated from both the host and the Arch container.

## Which to use

| You want…                                   | Use…                          |
| ------------------------------------------- | --------------------------- |
| A common desktop app, sandboxed             | Flatpak                      |
| Arch / AUR packages, dev toolchains         | `horizon-apps` (Distrobox)    |
| A point-and-click way to do either          | Horizon App Hub              |
| To change the base OS itself                | An [image update](image-updates.md) |
