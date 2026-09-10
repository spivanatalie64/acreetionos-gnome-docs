# Building the ISO

Horizon is built from the **ISO build profile** (an archiso profile) plus two
extra build steps that compile the patched desktop stack and the Freeman utility.

## Prerequisites

- An Arch Linux (or Arch-based) host with `archiso` / `mkarchiso` available.
- Root privileges (`mkarchiso` needs them).
- Enough disk space and RAM to compile GNOME 48 locally.
- A working network connection to fetch upstream sources on the first run.

## One-command build

```bash
./build.sh
```

This runs the full pipeline:

1. Builds the patched **XLibre + GNOME 48** desktop stack.
2. Builds the **Freeman** utility.
3. Constructs the bootable **Horizon ISO** with `mkarchiso`.

## Sub-steps

You can run the stages individually for module development or CI pipelining.

### Sub-step 1: XLibre desktop stack

```bash
./build-horizon-xlibre.sh
```

Pulls sources, applies the patches in `xlibre-patches/`, compiles, and injects the
packages into a local pacman repo that `mkarchiso` consumes.

### Sub-step 2: Freeman utility

```bash
./build-freeman.sh
```

Builds the Freeman CLI from the Rust sources in `tools/freeman/`.

### Sub-step 3: ISO assembly

Handled by `build.sh` (via `mkarchiso` / `mkarchiso.sh`) once the two steps above
have populated the local repository.

## Cleaning the workspace

```bash
./refresh.sh
```

Removes the `work/` and `out/` directories so the next build starts clean.

## Key configuration files

| File                        | Purpose                                                                 |
| --------------------------- | -------------------------------------------------------------------- |
| `profiledef.sh`             | archiso profile config: ISO metadata and file permissions.           |
| `packages.x86_64`           | Complete package list for the distribution.                          |
| `pacman.conf`               | Custom Pacman config used **only** to build the ISO.                 |
| `airootfs/`                 | Root filesystem overlay that becomes the live system.                |
| `airootfs/etc/calamares/`   | Bundled offline Calamares installer configuration.                   |
| `xlibre-patches/`           | Pinned source configs and patches for GNOME 48 / XLibre.             |
| `tools/freeman/`            | Rust source for the Freeman CLI.                                     |
| `grub/`, `syslinux/`, `efiboot/` | Bootloader configurations.                                     |

## Output

The finished ISO is written to the `out/` directory.
