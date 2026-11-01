# AGENTS.md — OpenMT2 project guidance

Global instructions for AI agents working in the OpenMT2 organisation.
Repository-local `AGENTS.md` files add build, test and environment details; when
they conflict with this file on a local matter, the local file wins.

## Read first

- **`open-mt2/devkit/docs/planning/windows-parity.md`** — the cross-platform,
  no-proprietary plan.
- **`open-mt2/devkit/docs/planning/windows-parity-tasks.md`** — the execution
  source of truth: task state, dependencies, size, verification command and
  evidence rules.
- **`open-mt2/docs`** — user-facing documentation.

## Repository-local guides

Build, test and environment details live in this repository, not in each
checkout:

- **devkit** — `docs/agents/repos/devkit.md` (image, build/test/smoke/golden
  commands, container rules, repository topology).

A repository may keep a short `AGENTS.md` that only points here so an agent
starting in that checkout finds the guide.

## Tracking

Every parity task is mirrored as an issue in `open-mt2/devkit`, titled
`[<ID>] <title>`, with one milestone per wave and the `parity` label. The task
sheet is the source of truth for scope and state; the issues are the progress
view and the place PRs close (`Closes #<n>`). Assign the issue to yourself when
you claim a task, and keep the sheet's checkbox, owner and counters in sync.

Blocked/undecided work is tracked in issue #1 (D1–D16). Deferred work carries
the `deferred` label.

## Hard rules

1. **Decisions gate tasks.** A task whose `deps` include an unchecked `D*`
   decision must not start. Ask the user; do not pick a decision silently.
2. **Test first.** No task lands without a test that fails before and passes
   after, or a recorded characterization artifact (render hash, pack transcript).
3. **One task per branch and per commit series.** Branch `task/<ID>-<slug>`,
   commit prefix `[<ID>]`.
4. **Nothing merged to `main` without review.** Push the branch, open a PR,
   report the link. Do not self-merge unless the user explicitly asks.
5. **Never commit generated or licensed content:** build outputs, fetched
   sources, caches, assets (`.gr2`, `.spt`, packs), or local scratch files.
6. **Structural changes are move-only.** File moves, renames and reformatting
   must be separate commits from behaviour changes.
7. **Evidence.** Every closed task writes `evidence/<ID>.md` with the command,
   before/after output, and notes.
8. **Keep scratch space off the checkout.** Generated files live under
   `.local`, which must be mapped to a temporary directory of the host. See
   [Local scratch space](#local-scratch-space).

## Local scratch space

`.local/` is the scratch area of a checkout: build logs, fetched artifacts,
temporary `dist/` trees and anything else generated for testing. Never keep it
on the repository filesystem — map `.local` to a temporary directory of the
host so it is cleaned on reboot and does not bloat the checkout or the Docker
build context.

Linux:

```bash
ln -s "$(mktemp -d "${TMPDIR:-/tmp}/openmt2-local.XXXXXX")" .local
```

macOS:

```bash
ln -s "$(mktemp -d "${TMPDIR:-/var/tmp}/openmt2-local.XXXXXX")" .local
```

Windows (`cmd`, no administrator rights needed — a junction, not a symlink):

```bat
mkdir "%TEMP%\openmt2-local" 2>nul
mklink /J .local "%TEMP%\openmt2-local"
```

Windows (PowerShell):

```powershell
$target = Join-Path $env:TEMP 'openmt2-local'
New-Item -ItemType Directory -Force $target | Out-Null
New-Item -ItemType Junction -Path .local -Target $target
```

Rules:

- Remove an existing real `.local` directory before creating the mapping; it
  only holds generated files.
- Each repository's `.gitignore` must contain `.local` **without a trailing
  slash**. A symlink or junction is not a directory to Git, so `.local/` does
  not ignore it.
- The mapping is per machine and is never committed.
- Docker mounts of `.local/...` follow the link on Linux and macOS. On Windows,
  pass the resolved `%TEMP%` path to `-v` if the junction is not followed.

## GitHub CLI

Use plain `gh` with the existing authentication. Check access with a bounded
read-only request before GitHub work:

```bash
timeout 20s gh api user --jq .login
gh repo view open-mt2/devkit
```

If the request succeeds, continue with `gh`. A missing `.local/gh` wrapper or
`GH_TOKEN` environment variable does not mean authentication is unavailable.
If authentication fails or times out, use `.local/gh` only when it exists in the
current checkout and `~/.config/openmt2/gh-token` is available; the wrapper
supplies `GH_TOKEN` to bypass the keyring. Otherwise report the actual error and
request authentication setup. Preserve `~/.config/gh/hosts.yml` unless the user
asks to change it.

Git: SSH works (`git@github.com` as `PoNo1101`). Remotes must use SSH, not HTTPS.

## Reporting

When a task closes, return exactly:

```
<ID> done.
changed: <paths>
tests: <n> added, <n> updated — `ctest` <pass/total>
evidence: evidence/<ID>.md
follow-ups: <new tasks or "none">
```

## Agent skills

### Issue tracker

Issues live in GitHub Issues on `open-mt2/devkit`. Follow the GitHub CLI
authentication policy above; see `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`,
`ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` plus `docs/adr/` at the repository root.
See `docs/agents/domain.md`.
