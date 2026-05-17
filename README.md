# edu-pkgbuild-icons

Batch build pipeline for Arch Linux icon packages — sardi, surfn, neo-candy and more — published to the [nemesis_repo](https://github.com/erikdubois/nemesis_repo) custom pacman repository.

## What this repo contains

Each subdirectory is a self-contained AUR-style package folder with its own `PKGBUILD` and build script. The root-level scripts orchestrate building all packages in one shot and publishing the repo.

**Icon families included:**

- **sardi** — circular icon sets with colora, flat, flexible, ghost, mono, mixing, mint-y and orb variants
- **surfn** — surfn icon sets including arc-breeze, mint-y and plasma themes
- **neo-candy** — neo candy icon set

## Prerequisites

- Arch Linux
- `devtools` package (`makechrootpkg`, `arch-nspawn`)
- A clean chroot at `~/Documents/chroot-archlinux` — create with:
  ```bash
  mkarchroot ~/Documents/chroot-archlinux/root base-devel
  ```
- `nemesis_repo` cloned to `~/EDU/nemesis_repo/`

## Usage

### Build all packages and publish the repo

```bash
./1-build-all-packages.sh
```

Iterates every package subdirectory, runs its build script, then calls `~/EDU/nemesis_repo/up.sh` to update the repository database.

### Build a single package

```bash
cd sardi-colora-variations-icons-git
./build.sh
```

The build script will:
1. Pull the latest upstream source (if it is a git repo)
2. Bump `pkgver`/`pkgrel` in the `PKGBUILD` (date-versioned packages only)
3. Skip the build entirely if the version has not changed since the last run
4. Build in the clean chroot under `/tmp/tempbuild/`
5. Copy the resulting `.pkg.tar.zst` to `~/EDU/nemesis_repo/x86_64/`

### Propagate the root build script to all packages

```bash
./copy-files-to-all-folders.sh
```

Copies the root-level `build.sh` to every package subdirectory that already has one, keeping them all in sync.

## Version scheme

Date-versioned packages (those with `pkgver` in `YY.MM` format) are bumped automatically:

- New month → `pkgver=YY.MM`, `pkgrel=01`
- Same month → `pkgrel` incremented (`02`, `03`, …)

Git-sourced packages (upstream-versioned) are left untouched by the bumper.

## Websites

- Information: https://erikdubois.be
- YouTube: https://www.youtube.com/erikdubois
