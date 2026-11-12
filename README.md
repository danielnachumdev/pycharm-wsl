# pycharm-wsl

Manage **JetBrains PyCharm** inside **WSL2**: list remote/local versions, install side-by-side, activate one, uninstall others.

Documents a manual WSL setup: tarball under `~/downloads`, zsh alias, WSLg vmoptions fix, Windows shortcut via `wslu`.

## Commands

```bash
./pycharm-wsl list
./pycharm-wsl install 2026.3              # download + activate (runtime fixes included)
./pycharm-wsl install 2026.2 --no-activate
./pycharm-wsl activate 2026.1             # switch active version (try / compare)
./pycharm-wsl uninstall 2026.1
```

`list` marks `(installed)` and `(active)` so you can verify local detection against the JetBrains catalog.

### Install vs activate (runtime fixes)

| Concern | Where |
| --- | --- |
| Download / extract / checksum | `install` |
| Symlink `~/downloads/pycharm`, zsh alias, WSLg `-Dsun.awt.wl.Shadow=false`, `wslu` shortcut | `activate` |

`install VERSION` runs `activate` afterward by default (one-shot). Use `--no-activate` to keep the current active version while adding another build for later trial.

## Layout

```text
~/downloads/pycharm-2026.3/     # versioned extract
~/downloads/pycharm -> …        # stable symlink = active version
~/downloads/pycharm-*.tar.gz    # cached archives
```

IDE settings under `~/.config/JetBrains/PyCharm*` are kept across switches; only the `wl.Shadow` VM option is ensured.

## What this encodes

| Piece | Detail |
| --- | --- |
| Install | versioned tarball extract under `~/downloads` |
| Launch | zsh alias → stable symlink `~/downloads/pycharm` |
| WSLg fix | `-Dsun.awt.wl.Shadow=false` in `pycharm64.vmoptions` |
| Shortcut | `wslusc -n "PyCharm WSL" …` |
| Browser | `BROWSER=wslview` |
| WSL | `/etc/wsl.conf`: `systemd=true`, `appendWindowsPath=false` (not rewritten by script) |

## Env

| Variable | Default |
| --- | --- |
| `PYCHARM_PRODUCT_CODE` | `PCP` (Professional; use `PCC` for Community) |
| `PYCHARM_INSTALL_ROOT` | `~/downloads` |
| `PYCHARM_STABLE_LINK` | `$INSTALL_ROOT/pycharm` |
| `PYCHARM_LIST_LIMIT` | `30` |
