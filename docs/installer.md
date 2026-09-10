# Calamares Installer

Horizon bundles a custom **Calamares** configuration under
`airootfs/etc/calamares/`, tuned for **offline installation** from the live ISO.

## What is customized

- **Offline by design.** The `dummy` package-operations module is used so the
  installer does not attempt an online package sync during installation.
- **Display stack defaults.** GDM and the X11 session defaults are preconfigured so
  the installed system boots straight into the GNOME 48 / XLibre session.
- **Post-install hooks.** The bundled post-install scripts and autostart hooks are
  wired up (including the [first-boot onboarding](freeman.md#first-boot) dialog).

## Filesystem

Installs target **ext4** partitions. Horizon relies on ext4 for stability and pairs
it with [Timeshift snapshots](recovery.md) rather than a copy-on-write filesystem.

## After installation

On first login the system shows the Horizon onboarding dialog explaining:

1. `pacman` is removed — do not re-add the Arch repositories.
2. [Freeman](freeman.md) handles update checks and image updates.
3. [`horizon-apps`](applications.md) provides an Arch container for extra software.
4. Flatpak is available and pre-configured.

The dialog writes `~/.config/horizon/.onboarded` and does not appear again.
