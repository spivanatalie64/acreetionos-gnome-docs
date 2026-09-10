# Freeman

**Freeman** (`/usr/local/bin/freeman`) is Horizon's customized package-query and
image-update tool. It replaces `pacman` for the things end users still need:
checking what is installed, and updating the system.

Freeman queries release metadata directly from **GitLab / GitHub releases**,
verifies cryptographic integrity, and stages updates locally before they are
applied to an inactive A/B root partition.

## Common commands

```bash
# Check if a new verified system image is available
freeman checkupdates

# Download and verify the update image locally (SHA-256)
freeman update

# Apply the staged update to the inactive A/B root partition
freeman update --apply
```

| Command                   | What it does                                                             |
| ------------------------- | --------------------------------------------------------------------- |
| `freeman checkupdates`    | Fetches release metadata and reports whether a newer image exists.    |
| `freeman update`          | Downloads the update image and verifies its SHA-256.                  |
| `freeman update --apply`  | Writes the verified, staged image to the inactive root and marks it for next boot. |

!!! tip
    `freeman checkupdates` and `freeman update` are safe to run at any time — they
    never touch the running system. Only `--apply` changes the boot target, and it
    writes to the **inactive** root, so the current system stays intact until you
    reboot.

## How it fits together

Freeman is the user-facing front end. The heavy lifting — downloading, verifying,
and staging the image — is done by the
[`horizon-image-update`](image-updates.md) backend, which Freeman invokes.

```
freeman ──► horizon-image-update ──► release metadata (GitLab/GitHub)
                     │
                     ├─ download image
                     ├─ verify SHA-256 (+ signature if configured)
                     └─ stage to inactive A/B root
```

## First boot

On first boot, `horizon-firstboot.sh` shows an onboarding dialog that explains
Freeman and the removal of `pacman`, so users understand how the system is
maintained. It writes a marker at `~/.config/horizon/.onboarded` and does not show
again.
