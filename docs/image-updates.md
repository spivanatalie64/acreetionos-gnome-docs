# Image-Based Updates

Horizon updates the OS as **full, verified, atomic system images** rather than
rolling package upgrades. This is what makes the pinned GNOME 48 / XLibre stack
safe to ship.

The backend is `/usr/local/bin/horizon-image-update`; [Freeman](freeman.md) is the
front end most users interact with. A systemd service and timer
(`horizon-image-update.service` / `.timer`) can run checks on a schedule.

## Update flow

1. **Fetch metadata.** Download the release manifest — either from a configured
   `UPDATE_MANIFEST_URL` or from the GitLab releases API for the configured
   project.
2. **Compare.** Determine whether the advertised image is newer than the running
   system on the selected channel.
3. **Download.** Pull the image into the state directory's `staged/` folder with
   retries.
4. **Verify.** Check the image against its **SHA-256**. If an
   `UPDATE_PUBLIC_KEY` is configured, the signature is checked too.
5. **Stage.** Keep the verified image and a `manifest.json` in the state
   directory.
6. **Apply.** On `freeman update --apply`, write the staged image to the
   **inactive** A/B root partition and mark it for the next boot.
7. **Reboot** to switch roots. The previous root remains as a fallback.

## Configuration

Settings live in `/etc/horizon/update.conf` (sourced by the backend). Defaults:

| Variable              | Default                                       | Meaning                                             |
| --------------------- | --------------------------------------------- | ------------------------------------------------- |
| `GITLAB_INSTANCE`     | `https://gitlab.acreetionos.org`             | GitLab instance hosting the releases.              |
| `GITLAB_PROJECT`      | `acreetionos-code/acreetionos-gnome`         | Project whose releases are polled.                 |
| `UPDATE_MANIFEST_URL` | *(empty)*                                     | If set, used instead of the GitLab releases API.   |
| `UPDATE_CHANNEL`      | `stable`                                      | Release channel to track.                          |
| `UPDATE_STATE_DIR`    | `/var/lib/horizon-image-update` (or per-user under `$XDG_DATA_HOME`) | Where manifests and staged images are kept. |
| `UPDATE_TARGET`       | *(empty)*                                     | Target block device / root slot for `--apply`.    |
| `UPDATE_PUBLIC_KEY`   | *(empty)*                                     | Public key for signature verification.             |

!!! note
    If `/var/lib` is not writable (for example, when invoked as a normal user), the
    backend falls back to a per-user state directory under
    `${XDG_DATA_HOME:-$HOME/.local/share}/horizon-image-update`.

## State directory contents

| Path                        | Contents                                  |
| --------------------------- | --------------------------------------- |
| `manifest.json`             | The most recently fetched release manifest. |
| `staged/`                   | Downloaded, verified update image(s).      |

## Scheduled checks

`horizon-image-update.timer` triggers `horizon-image-update.service` periodically
so `checkupdates` runs in the background. Applying an update always remains an
explicit action.
