# devkit — repository-local agent guide

Local instructions for AI agents working in
[`open-mt2/devkit`](https://github.com/open-mt2/devkit). Read the
[global rules](../../../AGENTS.md) first; this file only records what is
specific to this repository.

## What this repository is

The Docker development kit for the OpenMT2 client: image definition, build and
packaging scripts. The three client repositories are fetched at image build
time into `/workspace/src` and are **not tracked here**:

| Source | Destination |
|---|---|
| `open-mt2/neoeter` | `/workspace/src/neoeter` |
| `open-mt2/neoeter-python` | `/workspace/src/neoeter-python` |
| `open-mt2/client` | `/workspace/src/client` |

```
dockerfile                  image definition
docker/                     entrypoint + /workspace scaffold + CMake plumbing
scripts/build.sh            Linux build + dist packaging
scripts/build-windows.sh    Windows cross-build + dist packaging
scripts/smoke.sh            headless boot smoke (Linux + Wine)
scripts/fetch-sources.sh    clone/update the three source repositories
scripts/fetch-bin.sh        original client assets
scripts/fetch-windows-python.sh  vendored Windows CPython 2.7
scripts/bake-trees.sh       SpeedTree geometry cache baker (needs Wine)
scripts/golden-check.sh     render/API/transcript baselines
scripts/sources.map         source repositories and their order
docs/feature-matrix.md      generated fixed-function matrix (checked by ctest)
golden/                     pinned render/format/test baselines (tracked)
parity/ (agent-docs)        historical plan, task sheet and evidence
src/                        materialized sources (gitignored)
bin/                        client assets (gitignored, ~2.3 GB)
build/ dist/ treecache/     generated (gitignored)
```

## Read first

1. [`parity/docs/planning/windows-parity.md`](../../../parity/docs/planning/windows-parity.md)
   — the archived cross-platform / no-proprietary plan.
2. [`parity/docs/planning/windows-parity-tasks.md`](../../../parity/docs/planning/windows-parity-tasks.md)
   — the archived execution sheet: 38 tasks, 16 decisions, per-task size,
   dependencies, verification command and evidence rules. Wave B is closed; the
   only open scope is the deferred Wave 5. The sheet is historical — do not
   edit it for new work.

The per-task output is archived under `parity/evidence/` for history. New work
still uses GitHub issues (see below); there is no live task sheet anymore.

## Tracking

New work is tracked as an issue in `open-mt2/devkit`, titled `[<ID>] <title>`,
with the `parity` label. The historical task sheet in `parity/` records the
closed Waves A–B, 0–4 and the deferred Wave 5; the issues are the progress view
and the place PRs close (`Closes #<n>`). Assign the issue to yourself when you
claim it.

## Build and test

Populate `src/` once with `scripts/fetch-sources.sh` (the image does this at
build time), then use the image:

```bash
scripts/fetch-sources.sh                 # src/{neoeter,neoeter-python,client}

docker run --rm --user "$(id -u):$(id -g)" -e HOME=/tmp \
  -v "$PWD/src:/workspace/src" \
  -v "$PWD/build:/workspace/build" \
  -v "$PWD/dist:/workspace/dist" \
  openmt2/client -t --skip-pack
```

Rules for container runs:

- always `--user "$(id -u):$(id -g)" -e HOME=/tmp`, otherwise `dist/` and
  `build/` become root-owned;
- always mount `build/` so runs are incremental;
- vendor markers: `i686-w64-mingw32-gcc`, `x86_64-w64-mingw32-gcc/g++`,
  Wine 8.0 (32- and 64-bit), Python 2.7, CMake 3.25, Ninja, Xvfb + ImageMagick.

## Verification commands

```bash
# Linux build + full test suite (49 CTest cases, all green)
docker run ... openmt2/client -t --skip-pack          # see command above

# Windows cross-build + boot smoke
docker run ... openmt2/client windows
scripts/smoke.sh windows && scripts/smoke.sh linux

# cross-compile probe
docker run ... openmt2/client shell -c 'x86_64-w64-mingw32-g++ --version'

# golden baseline (11 checks, all green)
scripts/golden-check.sh

# source refresh
scripts/fetch-sources.sh

# repository state
git status --short && git log --oneline -3
```

## Never commit

`src/`, `bin/`, `build/`, `dist/`, `dist-linux/`, `dist-windows/`, `treecache/`,
`.local/`, and game assets (`.gr2`, `.spt`, packs). They are gitignored — do not
force-add them.

## Repository topology

`D14` split the client into three repositories:

- `neoeter` — engine (`include/Neoeter/...`, `src/{platform,io,image,render,audio,model,terrain,effect}`);
- `neoeter-python` — CPython 2.7 host and bindings (`include/Neoeter/Python/...`);
- `client` — game layer (`include/Client/...`, `src/{game,ui,webbrowser}`).

The 16 `pkg_*` repositories were merged with their history, tagged `final`,
mirrored under `../archive/` and deleted. Edit the three repositories, never
create per-library repositories.
