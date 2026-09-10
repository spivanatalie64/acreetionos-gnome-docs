# AcreetionOS Horizon

**AcreetionOS Horizon** is the Arch Linux based **community desktop edition**,
targeting x86_64 and delivering a native **X11 GNOME 48** desktop powered by
**XLibre**. It completely replaces `pacman` with the **Freeman** update utility for
Android / SteamOS-style atomic image updates.

Wayland is deliberately **not** the target for Horizon.

---

## Start here

<div class="grid cards" markdown>

- :material-book-open-variant: **[Overview](overview.md)** — what Horizon is, and why
  it drops `pacman`.
- :material-hammer-wrench: **[Building the ISO](building.md)** — the one-command build
  and its sub-steps.
- :material-package-variant-closed: **[Freeman](freeman.md)** — package queries and
  image updates.
- :material-cloud-download: **[Image-Based Updates](image-updates.md)** — how atomic
  updates are fetched, verified, and applied.
- :material-apps: **[Applications & App Hub](applications.md)** — installing and
  launching software without touching the CLI.
- :material-backup-restore: **[Recovery & Snapshots](recovery.md)** — Timeshift and
  the recovery environment.

</div>

## At a glance

| Aspect            | Horizon                                                        |
| ----------------- | ------------------------------------------------------------- |
| Base              | Arch Linux (x86_64)                                            |
| Desktop           | GNOME 48, patched for XLibre                                   |
| Display server    | XLibre (X11) — **not** Wayland                                 |
| Package manager   | Removed — replaced by **Freeman**                             |
| Updates           | Full, verified, atomic system images (A/B root)                |
| Filesystem        | ext4, with Timeshift (rsync) snapshots                        |
| Installer         | Bundled offline **Calamares**                                  |
| Extra apps        | Arch Linux **Distrobox** container + graphical App Hub        |
| License           | See [`LICENSE`](https://github.com/spivanatalie64/acreetionos-gnome-docs/blob/main/LICENSE) |

!!! note "Where the code lives"
    This site documents the **ISO build profile**. The build sources live on the
    AcreetionOS GitLab at <https://gitlab.acreetionos.org>. This documentation repo
    is mirrored to GitHub so Read the Docs can build it.
