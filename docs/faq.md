# FAQ

### Can I run `pacman -Syu`?

No. `pacman` and its keyring tools are removed. Re-adding the Arch repositories
will break the keyring and can render the desktop unbootable. Use
[Freeman](freeman.md) for system updates and [`horizon-apps`](applications.md) for
extra packages.

### Does Horizon support Wayland?

No. Horizon deliberately targets a **native X11** GNOME 48 session on **XLibre**.
Wayland is not a goal for this edition.

### How do I install an application?

Use the [Horizon App Hub](applications.md) or `horizon-apps` to install into the
Arch Distrobox container, or install a **Flatpak**. Exported container apps appear
in the normal GNOME menu with an `[Arch App]` badge.

### How do updates work?

The whole OS is replaced by a verified image. Freeman downloads and verifies it
(SHA-256, plus signature when configured), stages it, and — on
`freeman update --apply` — writes it to the inactive A/B root. You reboot to switch.
See [Image-Based Updates](image-updates.md).

### What if an update won't boot?

The previous root partition is still there. Boot it (or the recovery environment)
and restore a [Timeshift snapshot](recovery.md) or re-apply a known-good image.

### Where is the source?

The ISO build profile lives on the AcreetionOS GitLab at
<https://gitlab.acreetionos.org>. This documentation is maintained in a separate
repo mirrored to GitHub so Read the Docs can build it.

### How do I edit these docs?

Click the pencil icon at the top of any page — it links straight to the Markdown
source. Or clone the docs repo, run `mkdocs serve`, and open a merge/pull request.
