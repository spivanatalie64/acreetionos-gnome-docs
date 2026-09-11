# Architecture & Build Internals

This page goes one level deeper than [Building the ISO](building.md): how the
patched desktop stack is actually bootstrapped from source, and how Freeman is
put together.

## The staged GNOME build

Building `gnome-shell` from source requires `mutter`'s headers, GIR/typelib
files, and libraries — but `mutter` itself hasn't been installed anywhere yet.
`xlibre-patches/build-patched-packages.sh` solves this bootstrapping problem with
a **shared staging tree** that grows as each package finishes, rather than
relying on a system package manager to resolve the chain:

```text
.horizon-build/
├── sources/<pkg>/     # pinned git clone, patches applied
├── meson/<pkg>/       # meson build directory
├── stage/<pkg>/       # DESTDIR install of just this package
├── stage_usr/         # shared tree: headers, .gir/.typelib, mutter-16 libs
├── stage_pkgconfig/   # merged .pc files from every staged package so far
└── package/<pkg>/     # generated PKGBUILD + the final makepkg output
```

For every package in `xlibre-patches/packages.conf`, the script:

1. **Clones** the pinned tag and applies that package's patches from
   `xlibre-patches/<package>/*.patch`.
2. **Exports compiler/linker flags** pointing at everything staged so far —
   `CFLAGS`/`CPPFLAGS` get `-I` paths into `stage_usr/include` (including
   `mutter-16`'s `clutter`/`cogl`/`mtk` subdirs), `LDFLAGS`/`LD_LIBRARY_PATH`
   get `-L` paths into the previous package's `stage/*/usr/lib`, and
   `PKG_CONFIG_PATH` points at the merged `stage_pkgconfig/`.
3. **Builds with meson** (`meson setup` → `meson compile` → `DESTDIR=... meson
   install`) using that package's pinned flags from `packages.conf`.
4. **Normalizes the install** — merges any `sbin/` into `bin/` for Arch package
   layout, and drops `etc/gdm/custom.conf` from the `gdm` stage so the
   `airootfs` overlay can ship its own copy without a file conflict.
5. **Re-stages headers, GIR/typelib, and libraries** into `stage_usr/` —
   including exporting `XDG_DATA_DIRS` — so the *next* package's
   `g-ir-scanner` run (e.g. `gnome-shell` introspecting `Clutter-16`/`Meta-16`)
   can resolve `mutter`'s types even though `mutter` was never installed
   system-wide.
6. **Wraps the staged output in a minimal PKGBUILD** (`pkgdesc='AcreetionOS
   Horizon patched GNOME component'`) and runs `makepkg`, then adds the result
   to the local pacman repo in `.horizon-build/repo` with `repo-add`.

That local repo is what `mkarchiso` (via `packages.x86_64`) pulls the patched
GNOME 48 / XLibre packages from when assembling the ISO.

!!! note "Why not just use Arch's packages?"
    Arch's GNOME packages track upstream's Wayland-first direction. Horizon needs
    X11 forced on and Wayland/Xwayland forced off at the meson level
    (`-Dwayland=false -Dxwayland=false` for `mutter`, `-Dwayland-support=false`
    for `gdm`) — flags upstream Arch packaging doesn't set — which is why these
    four packages are rebuilt from pinned source instead of consumed as-is.

## Freeman's internals

Freeman is a small Rust **workspace**, not a single binary — `tools/freeman/`
has five crates:

| Crate            | Responsibility                                                         |
| ---------------- | ------------------------------------------------------------------- |
| `freeman-config` | Shared configuration types.                                           |
| `freeman-core`   | Shared core types/logic used by the other crates.                     |
| `freeman-db`     | Local package queries via **`alpm`** (the ALPM/libalpm bindings `pacman` itself is built on) — Freeman reads the package database without needing `pacman` present. |
| `freeman-aur`    | AUR lookups over HTTP (`ureq` + `serde_json`).                         |
| `freeman-cli`    | The command-line front end. |

The CLI crate's compiled binary is literally named **`pamac`** (see the
`[[bin]]` section of `freeman-cli/Cargo.toml`) — a deliberate drop-in name so
existing muscle-memory / scripts expecting a `pamac`-shaped package tool keep
working, while the `freeman` entry point documented in [Freeman](freeman.md) is
the supported user-facing command.

Release profile: the workspace builds with `lto = "thin"`,
`codegen-units = 1`, and `strip = true` — optimized for a small, fast static-ish
CLI rather than fast incremental rebuilds.

## Calamares + `horizon-installer`

The bundled Calamares installer is backed by custom Python modules under
`airootfs/usr/local/lib/horizon-installer/`:

- **`hardware_detect.py`** — auto-detects and preselects GPU/hardware drivers
  instead of asking the user to pick blindly.
- **`migration_backup.py` / `restore_data.py`** — detect an existing Windows,
  macOS, or Linux installation on the disk and preserve/restore user data across
  the install, rather than treating every install as a blank disk.
- **`installer.py`** — assembles the actual Calamares `settings.conf` job
  sequence for two install profiles:
  - **`offline`** — no `netinstall` jobs are queued at all; only the core
    sequence (`unpackfs@rootfs`, `initcpio`, `bootloader`, `umount`, …) runs, so
    a fully offline install can never get stuck waiting on the network.
  - **`online`** — exposes `netinstall@drivers` and `netinstall@software`
    package groups (each group visible, none hidden) and guarantees they run
    *before* the install summary, so extra drivers/software are chosen before
    committing to the install.

This is also configured for **zswap/zram** compatibility so the installed
system's swap setup doesn't fight with the live/installed environment's own
compressed-memory configuration.

## Verifying the build logic

`tests/test_features.py` and `tests/test_installer_profiles.py` exercise these
modules directly (loading `hardware_detect.py`, `migration_backup.py`,
`restore_data.py`, and `installer.py` and asserting on the generated Calamares
config) rather than only being checked by running the full ISO build —
`installer.prepare_profile(...)` is unit-testable without Calamares or a VM.
