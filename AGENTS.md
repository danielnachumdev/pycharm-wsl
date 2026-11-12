# pycharm-wsl – Agent Context

How this repo works. Product pitch: [`README.md`](README.md).

---

## Orientation

| Question | Answer |
| -------- | ------ |
| What is this? | Bash CLI to list / install / uninstall / activate JetBrains PyCharm under WSL2 |
| Entry point | `./pycharm-wsl` (executable script at repo root) |
| Default product | Professional (`PCP`); Community via `PYCHARM_PRODUCT_CODE=PCC` |
| Default install root | `~/downloads` (`pycharm-<version>/`, archives, stable `pycharm` symlink) |
| Remote catalog | JetBrains public releases API |
| Secrets / PII | None in repo; do not hardcode home layouts like `~/dev/...` |

```text
.
├── pycharm-wsl   # CLI
├── README.md     # WHAT
└── AGENTS.md     # HOW (this file)
```

---

## Commands

| Command | Behavior |
| ------- | -------- |
| `list` | Remote releases + local extracts; marks `(installed)` / `(active)` |
| `install VERSION [--no-activate]` | Download, sha256-check, extract. Runs `activate` unless `--no-activate` |
| `uninstall VERSION [--keep-archive]` | Remove extract (and archive unless `--keep-archive`). Clears symlink if that version was active |
| `activate VERSION` | Point stable symlink + zsh alias + Windows shortcut at VERSION; migrate plugins + recent projects + IDE settings; apply WSLg vmoptions; ensure `wslu` |

`list` is the detection check: if a local install is healthy it must show `(installed)`.

### Install vs activate

| Concern | Command |
| ------- | ------- |
| Download / extract / checksum | `install` |
| Symlink `~/downloads/pycharm`, zsh `pycharm` alias, plugins / recent projects / IDE settings, `-Dsun.awt.wl.Shadow=false`, `BROWSER=wslview`, `wslusc -g` desktop shortcut | `activate` |

### State migration (on `activate`)

| What | From → to | Policy |
| ---- | --------- | ------ |
| User plugins | `~/.local/share/JetBrains/<dataDirectoryName>/` | `rsync -a --ignore-existing` from richest prior dir |
| Recent projects | `…/options/recentProjects.xml` (+ `trusted-paths.xml`, `project.default.xml`) | copy if missing; replace if source has more `entry key=` rows |
| Workspace snapshots | `…/workspace/` | `rsync -a --ignore-existing` |
| IDE settings | SDKs (`jdk.table.xml`, `pySdk.xml`, `nodejs.xml`), editor/UI XML, GitHub/GitLab/DB, `disabled_plugins.txt`; dirs `keymaps/`, `codestyles/`, `inspection/`, `extensions/`, `grazie/`, `tasks/`, `ssl/` | files: copy if missing or source larger; dirs: `--ignore-existing` |
| Not migrated | caches/DBs, `user.web.token`, promo/survey XML, `bundled_plugins.txt`, `disabled_update.txt` | — |

`dataDirectoryName` comes from that version’s `product-info.json` (e.g. `PyCharm2026.3`). `install` runs `activate` by default, so a normal upgrade migrates this automatically.

Runtime hooks are idempotent and live in `activate` so version switches re-apply them.

---

## Layout on disk

```text
~/downloads/pycharm-2026.3/     # versioned extract
~/downloads/pycharm -> …        # stable symlink = active version
~/downloads/pycharm-*.tar.gz    # cached archives
```

IDE config under `~/.config/JetBrains/PyCharm*` is left intact across switches; activate only ensures the `wl.Shadow` VM option exists for relevant config dirs.

---

## WSL / Windows integration

Encoded operational knowledge (do not regress casually):

| Piece | Detail |
| ----- | ------ |
| Launch alias | `alias pycharm='( "$HOME/downloads/pycharm/bin/pycharm.sh" > /dev/null 2>&1 & )'` in `~/.zshrc` |
| WSLg fix | `-Dsun.awt.wl.Shadow=false` in `pycharm64.vmoptions` |
| Desktop shortcut | `wslusc -g` → **`$INSTALL_ROOT/pycharm-wsl-run`** (fixed wrapper that `exec`s `$STABLE_LINK/bin/pycharm.sh`). Needed because `wslusc` does `readlink -f` and would otherwise bake `pycharm-<version>` into the `.lnk`. Shell alias uses `$STABLE_LINK` directly. `-g` required for GUI. |
| Browser | `command -v wslview >/dev/null && export BROWSER=wslview` |
| Host WSL config | `/etc/wsl.conf` may use `systemd=true` and `appendWindowsPath=false` — **not** rewritten by this script |

---

## Environment

| Variable | Default |
| -------- | ------- |
| `PYCHARM_PRODUCT_CODE` | `PCP` |
| `PYCHARM_INSTALL_ROOT` | `$HOME/downloads` |
| `PYCHARM_STABLE_LINK` | `$INSTALL_ROOT/pycharm` |
| `PYCHARM_ZSHRC` | `$HOME/.zshrc` |
| `PYCHARM_SHORTCUT_NAME` | `PyCharm WSL` |
| `PYCHARM_LIST_LIMIT` | `30` |

Dependencies: `bash`, `curl`, `jq`, `tar`, `rsync`; on WSL activate may install `wslu` / `imagemagick` via `apt` (sudo).

Optional: `pv` — if present, downloads use `pv -p -t -e -r -b` (bytes + rate + ETA). Otherwise curl’s classic progress meter (%, size, time left, speed) on a TTY.

---

## Agent rules

- Keep README about **what** / why; put mechanics here.
- Prefer `~/downloads` defaults — no private path layouts in committed docs or script defaults.
- `activate` must keep `-g` on `wslusc` for GUI shortcuts.
- Do not wipe `~/.config/JetBrains` as part of install/activate.
- When changing version detection, verify with `./pycharm-wsl list` against a known local extract.
