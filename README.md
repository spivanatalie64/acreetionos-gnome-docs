# AcreetionOS GNOME (Horizon) Documentation

Source for the AcreetionOS Horizon documentation site, built with
[MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
and published on [Read the Docs](https://readthedocs.org/).

**Live site:** https://acreetionos-gnome.readthedocs.io/

AcreetionOS Horizon is the Arch Linux based community desktop edition built around a
maintained **GNOME 48 + XLibre (X11)** stack, with `pacman` removed in favour of
**Freeman** and Android/SteamOS-style atomic image updates.

## Local preview

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000/.

## Build a static site

```bash
mkdocs build          # outputs to ./site
```

## Layout

| Path                 | Purpose                                          |
| -------------------- | ----------------------------------------------- |
| `mkdocs.yml`         | Site config, theme, and navigation              |
| `docs/`              | Markdown content, one file per nav entry        |
| `.readthedocs.yaml`  | Read the Docs build configuration (v2)           |
| `requirements.txt`   | Python build dependencies (pinned)              |

## Contributing

Edit the relevant file under `docs/`, run `mkdocs serve` to check it, and open a
merge/pull request. Every page has an "edit" pencil linking straight to its source.

The upstream project (ISO build profile) lives on the AcreetionOS GitLab:
<https://gitlab.acreetionos.org>.

## License

See [`LICENSE`](LICENSE).
