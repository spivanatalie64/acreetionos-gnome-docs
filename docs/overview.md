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
| **Calamares + `horizon-installer`** | Bundled offline installer with hardware detection and OS migration support. |
| **Distrobox + App Hub**          | Safe place to install and launch extra applications.                 |
| **Timeshift**                    | rsync snapshots on ext4 for rollback.                                |

For how the pieces above are actually built — the staged GNOME compile, Freeman's
Rust workspace, and the exact patched package versions — see
[Architecture & Build Internals](architecture.md).

## Pinned desktop versions

The desktop stack is not "whatever GNOME 48.x Arch ships this week" — it is a
specific, pinned set of sources (`xlibre-patches/packages.conf`) built with X11
forced on and Wayland/Xwayland forced off:

| Package           | Pinned tag | Notable meson flags                                   |
| ------------------ | ---------- | ------------------------------------------------------ |
| `mutter`           | `48.8`     | `-Dx11=true -Dwayland=false -Dxwayland=false`           |
| `gnome-session`    | `48.0`     | `-Dx11=true`                                            |
| `gdm`              | `48.0`     | `-Dx11-support=true -Dgdm-xsession=true -Dwayland-support=false` |
| `gnome-shell`      | `48.8`     | `-Dtests=false`                                         |

Each gets small, targeted downstream patches (for example, a Pango-renderer
`autoptr` fix for `mutter`, an `ical` const-signature fix for `gnome-shell`, and a
`stdbool` header fix for `gdm`) rather than wholesale forks — the goal is the
smallest possible diff that keeps a native X11 session building against current
GNOME 48 sources.

## Design principles

- **Immutable display stack.** The GNOME/XLibre binaries are pinned; nothing on the
  running system rewrites them piecemeal.
- **Atomic, verified updates.** Every update is a full image with a SHA-256 (and,
  where configured, signature) check before it is applied.
- **A/B roots.** Updates are written to the inactive root and activated on reboot,
  so a bad image never bricks the current system.
- **No CLI required.** Installing apps, launching them, and taking snapshots are all
  possible from graphical tools.
