# Recovery & Snapshots

Horizon runs on **ext4** and protects the system with **Timeshift** snapshots plus
a dedicated **recovery environment**.

## Timeshift snapshots

- **rsync** snapshots run on boot and on a daily schedule.
- The system root and persistent user data are isolated, so image updates and app
  experiments cannot cause data loss.
- If a configuration change or corruption breaks the system, Timeshift restores the
  root filesystem to a previous known-good state.
- Snapshots can also be triggered on demand from the
  [Horizon App Hub](applications.md).

## A/B roots

Image updates are written to the **inactive** root partition and only become active
on reboot. The previous root stays in place as an automatic fallback if a new image
fails to boot. See [Image-Based Updates](image-updates.md).

## Recovery environment

`acreetion-recovery` is the master recovery launcher. It:

- Probes the hardware and selects a **GUI** (`acreetion-recovery-gui`) or **TUI**
  (`acreetion-recovery-tui`) front end, with fallback in both directions.
- Ensures the root filesystem is mounted read-write before starting.
- Uses the backend at `/usr/local/lib/acreetion/recovery-backend.sh`.

### Flags

| Flag              | Effect                                            |
| ----------------- | ---------------------------------------------- |
| `--tui`, `-t`     | Force the text (TUI) interface.                 |
| `--gui`, `-g`     | Force the graphical (GUI) interface.            |
| `--scan`, `-s`    | Run a hardware/system scan and exit.            |

## Recovery checklist

1. Boot into the recovery environment (or the fallback A/B root).
2. Run `acreetion-recovery --scan` to inspect the system.
3. Restore the most recent good **Timeshift** snapshot, **or**
4. Re-apply / roll back a system image with [Freeman](freeman.md).
5. Reboot.
