# OpenMT2 agent docs

Global guidance for AI agents working across the OpenMT2 repositories.

This repository was split out of [`open-mt2/devkit`](https://github.com/open-mt2/devkit)
in October 2026. The guidance is organisation-wide — tracking, branch and PR
rules, evidence, GitHub authentication, reporting — and was cluttering a build
repository that contains no client sources.

| Path                     | Purpose                                                              |
| ------------------------ | -------------------------------------------------------------------- |
| `AGENTS.md`              | Global agent instructions. Repository-local `AGENTS.md` files add build, test and environment details. |
| `docs/agents/`           | Agent skills: issue tracker, triage labels, domain docs.             |
| `docs/agents/repos/`     | Repository-local guides (build, test and environment details).       |
| `scratch/issue-review/`  | Archived AI-generated issue rewrite drafts (October 2026). Historical, not current guidance. |

Imported from `open-mt2/devkit@023fb8a` (paths `AGENTS.md`, `docs/agents/` and
`.scratch/issue-review/`). History before that commit remains in the devkit
repository.

The devkit-local guide was moved here in October 2026
(`docs/agents/repos/devkit.md`); the devkit checkout keeps a short pointer.

## Using it

Read `AGENTS.md` before working in any OpenMT2 repository. Repository-local
guides live under `docs/agents/repos/`; a checkout may keep a short `AGENTS.md`
that only points here. For devkit build and container commands read
[`docs/agents/repos/devkit.md`](docs/agents/repos/devkit.md).
