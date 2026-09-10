# Overview

## Why Horizon exists

Modern upstream desktop releases are moving aggressively toward Wayland-only
sessions. Horizon retains a **native X11 session** by building against the
**GNOME 48** source release, patched locally for full **XLibre** support and
stability.

The core desktop packages are compiled at build time by
[`./build-horizon-xlibre.sh`](building.md#sub-step-1-xlibre-desktop-stack), which:

1. Pulls the GNOME 48 / XLibre sources.
2. Applies the downstream patches in `xlibre-patches/`.
3. Compiles them.
4. Injects the results into a local pacman repository for `mkarchiso` to consume.

## Why `pacman` is completely removed

In an immutable, pinned desktop environment, running standard rolling upgrades
with `pacman -Syu` against live Arch mirrors will break system compatibility. It
would replace the patched GNOME 48 / XLibre binaries with Wayland-only or
ABI-incompatible versions, leaving the desktop unable to start.

Therefore:

- `pacman` and its key-management tools are **removed** from the installed OS.
- Re-adding the Arch repositories will break the keyring — **do not do it**.
- System updates ship as **full, verified, atomic system images** (similar to
  Android or SteamOS) rather than file-by-file package swaps.

!!! warning
    Because there is no host package manager, system-wide software changes happen
    only through image updates. Day-to-day apps go in the
    [Arch Distrobox container](applications.md) instead.

## The pieces

| Component                        | Role                                                                 |
| -------------------------------- | ------------------------------------------------------------------- |
| **GNOME 48 + XLibre**            | The desktop, built from source with downstream patches.              |
| **Freeman** (`/usr/local/bin/freeman`) | Package queries + image-update client.                        |
| **`horizon-image-update`**       | Backend that downloads, verifies, and stages update images.          |
| **`horizon-firstboot.sh`**       | First-boot onboarding dialog explaining Freeman and the `pacman` removal. |
| **Calamares**                    | Bundled offline installer under `airootfs/etc/calamares/`.           |
| **Distrobox + App Hub**          | Safe place to install and launch extra applications.                 |
| **Timeshift**                    | rsync snapshots on ext4 for rollback.                                |

## Design principles

- **Immutable display stack.** The GNOME/XLibre binaries are pinned; nothing on the
  running system rewrites them piecemeal.
- **Atomic, verified updates.** Every update is a full image with a SHA-256 (and,
  where configured, signature) check before it is applied.
- **A/B roots.** Updates are written to the inactive root and activated on reboot,
  so a bad image never bricks the current system.
- **No CLI required.** Installing apps, launching them, and taking snapshots are all
  possible from graphical tools.
