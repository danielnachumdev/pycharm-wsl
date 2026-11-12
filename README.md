# pycharm-wsl

**PyCharm on WSL, without getting stuck on one tarball forever.**

Running JetBrains PyCharm inside WSL2 usually means downloading a Linux archive, unpacking it somewhere, and wiring a shell alias or desktop shortcut by hand. That works once — then upgrades are awkward, side-by-side versions are messy, and WSLg quirks show up as “it just won’t open.”

**pycharm-wsl** turns that into a small CLI:

- See what’s available from JetBrains and what’s already on disk
- Install any version you pick
- Keep several versions side by side
- Activate the one you want (symlink, shell launch, Windows shortcut, WSLg fixes)
- Uninstall versions you no longer need

Built for people who run PyCharm **from Linux inside WSL**, not the Windows installer.

```bash
./pycharm-wsl list
./pycharm-wsl install 2026.3
./pycharm-wsl activate 2026.1   # try another build without throwing the old one away
```

Implementation details for agents and maintainers: [`AGENTS.md`](AGENTS.md).
