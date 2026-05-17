# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo is a batch build pipeline for Arch Linux icon packages (`sardi-*`, `surfn-*`, `neo-candy-*`) destined for the `nemesis_repo` custom pacman repository. Each subdirectory is a self-contained AUR-style package folder containing a `PKGBUILD`, a `build-icons.sh` (or `build*.sh`) script, and version-tracking dot-files.

## Key scripts

| Script | Role |
|---|---|
| `build.sh` | Per-package build script: bumps version, checks if build is needed, builds in chroot (`~/Documents/chroot-archlinux`) via `makechrootpkg`, copies `.pkg.tar.zst` to `~/EDU/nemesis_repo/x86_64/` |
| `1-build-all-packages.sh` | Iterates every subdirectory, runs its `build*` script, then calls `~/EDU/nemesis_repo/up.sh` to publish the repo |
| `copy-files-to-all-folders.sh` | Propagates the root-level `build.sh` to every package subdirectory that already has one (repo-name detection: copies `build.sh` when repo name contains `pkgbuild`) |
| `91-copy-build-data-to-all-folders.sh` | Older helper — copies `build-icons*` into every subdirectory (legacy, less structured) |
| `setup.sh` | Configures the git remote to `git@github.com-edu:erikdubois/<project>` (SSH host alias) |
| `up.sh` | Ensures git remote, pulls, optionally runs `chaotic.sh`/`repo.sh`, then commits and pushes |

## Version scheme

Packages whose `pkgver` matches `YY.MM` (e.g. `26.03`) are **date-versioned** and are bumped automatically by `build.sh`:
- New month → `pkgver=YY.MM`, `pkgrel=01`
- Same month → `pkgrel` incremented (zero-padded, e.g. `02`, `03`)

Packages with upstream-style versions (e.g. git packages) are skipped by the bumper.

The build is skipped entirely when `.current-version` matches `.previous-version` (no change detected).

## Per-package structure

```
<package-name>/
  PKGBUILD            # Arch package descriptor
  build-icons.sh      # Older per-package build script (legacy variant)
  build.sh            # Modern per-package build script (copied from root)
  .current-version    # Written at start of each build run
  .previous-version   # Copied from .current-version after successful build
  .SRCINFO            # Generated; not hand-edited
  .Changelog          # Optional per-package changelog
```

## Build environment

- Chroot lives at `~/Documents/chroot-archlinux` — must be initialised with `mkarchroot` before first use.
- Built packages land in `~/EDU/nemesis_repo/x86_64/`.
- Build artefacts are staged under `/tmp/tempbuild/` (cleaned before each run).
- Failures are logged to `/tmp/failed`; packages with >2 old versions in the repo are noted in `/tmp/installed`.

## Bash script conventions

All scripts in this repo follow the standard template (defined in global CLAUDE.md):
- `#!/bin/bash` + `set -euo pipefail`
- Standard header block
- `SCRIPT_DIR` via `${BASH_SOURCE[0]}`
- tput color block with TTY fallback
- `log_section` / `log_info` / `log_warn` / `log_error` / `log_success` functions
- `on_error` + `trap … ERR`
- `main()` ending with `log_success "$(basename "$0") done"`

The older `build-icons.sh` files inside package subdirectories predate this template and do not conform to it — do not use them as a style reference.
