<div align="center">
  <h1>awecontrib</h1>
  <p><strong>One verify entry per repo, one version truth per repo.</strong></p>
  <p>
    <strong>English</strong> ·
    <a href="./README_cn.md">简体中文</a>
  </p>
  <p>
    <img src="https://img.shields.io/pypi/v/awecontrib?style=flat-square&color=7C3AED" alt="Version">
    <img src="https://img.shields.io/badge/python-%E2%89%A53.9-0EA5E9?style=flat-square" alt="Python">
    <img src="https://img.shields.io/badge/license-MPL--2.0-22C55E?style=flat-square" alt="License">
  </p>
  <p>
    <img src="https://img.shields.io/badge/status-alpha-c96a3d?style=flat-square" alt="Status">
    <img src="https://img.shields.io/badge/install-pip-22C55E?style=flat-square" alt="pip install">
  </p>
</div>

> `awecontrib install` drops a small `verify` file and a minimal CI into a repo. Local and CI then run the exact same command, so they can never drift apart.

## Why

Repos drift the same way everywhere: local development runs `pytest` while the release workflow runs `unittest discover`, tests exist but no CI ever calls them, the version lives in two places depending on the file, and `egg-info`/`__pycache__` junk quietly gets committed. The fix is not more process — it is one entry point per repo that both you and CI call.

awecontrib grew out of the [awe series](#in-use-the-awe-series), a family of CLI and desktop tools that hit every one of these problems as it grew repo by repo. The tool keeps the lessons and drops the family coupling: nothing it writes depends on awecontrib or any naming convention, so any Python or Node repo can use it.

## Install

```bash
pip install awecontrib
```

### Agent skill

The repo ships a companion skill at `resources/skills/awecontrib` so coding agents know when to reach for this CLI:

```bash
aweskill install wehuman01/awecontrib
```

## Commands

### `awecontrib install`

Run at the repo root. Detects Python (pyproject.toml) vs Node (package.json); pass `--python` or `--node` when both exist.

- Python repo: writes an executable `verify` (pytest, plus `ruff check .` only if the repo configures ruff), a 6-line `.github/workflows/ci.yml` that just calls `./verify`, and appends junk patterns to `.gitignore`.
- Node repo: composes `scripts.verify` from the typecheck/lint/test scripts that already exist in package.json, writes the CI workflow (npm/pnpm/yarn detected from the lockfile), and appends `.gitignore` entries.

Never overwrites an existing `verify` or CI file without `--force`.

### `awecontrib bump <version>`

Sets the version in the one place it lives — static `version` in pyproject, `__version__` in `src/*/__init__.py` for dynamic versioning, or package.json — and prepends a `## v<version>` entry to `docs/CHANGELOG.md` (body from `--note`). No git commits or tags; your release flow keeps owning those.

### `awecontrib hygiene [--fix]`

Fails if junk files (`egg-info`, `__pycache__`, `.pytest_cache`, `.DS_Store`, `*.pyc`, `node_modules`) are tracked in git. `--fix` prints the exact `git rm -r --cached` command instead of running it.

## The verify file

Self-contained bash — CI does not need awecontrib installed:

```bash
#!/usr/bin/env bash
set -e
cd "$(dirname "$0")"

PY=".venv/bin/python"        # prefer the repo venv; CI falls back to python3
[ -x "$PY" ] || PY="python3"

"$PY" -m pytest

# hygiene: tracked junk files fail the build
JUNK_RE='(\.egg-info/|__pycache__/|\.pytest_cache/|\.DS_Store$|\.pyc$|^node_modules/)'
...
```

Edit it freely; it is yours. If your release workflow still runs `unittest discover`, replace that step with `./verify` so publishing uses the same checks as development.

## In use: the awe series

Every repo in the [awe series](https://github.com/wehuman01) runs through the same `./verify` gate — the tool is maintained against that fleet first, so each project doubles as a live case study:

| Repo | What it is | Stack |
|---|---|---|
| [awerouter](https://github.com/wehuman01/awerouter) | LLM proxy that routes coding-agent traffic between flash and pro models | Python |
| [awecompress](https://github.com/wehuman01/awecompress) | Context-compression proxy that freezes old conversation turns into a cached summary | Python |
| [awewarm](https://github.com/wehuman01/awewarm) | Keeps AI coding subscription windows warm on a schedule | Python |
| [aweswitch](https://github.com/wehuman01/aweswitch) | Switches API profiles for Claude Code, Codex, OpenCode, and ZCode | Python |
| [awescholar](https://github.com/wehuman01/awescholar) | AI-operable scholarly literature search and curation CLI | Python |
| [aweshelf](https://github.com/wehuman01/aweshelf) | Bookmarks, categorizes, and resumes AI coding sessions | Python |
| [aweskill](https://github.com/wehuman01/aweskill) | Package manager that projects agent skills into 47+ agent directories | Node/TypeScript |
| [awehitch](https://github.com/wehuman01/awehitch) | Uses ChatGPT web as the planning brain for local coding agents over MCP | Node/TypeScript |
| [aweshare](https://github.com/wehuman01/aweshare) | Local-first relay that shares spare Ollama/vLLM capacity through one hub | Node/TypeScript |
| [awefork](https://github.com/wehuman01/awefork) | Desktop workbench for forking AI coding sessions at any turn | Electron/TypeScript |
| [awedot](https://github.com/Webioinfo01/awedot) | Desktop floating ball that bookmarks and restores coding sessions | Tauri/Rust + TypeScript |

The family also curates [AgentX](https://github.com/Webioinfo01/agentx-hub) — a community directory of scientific research AI agents, run on awescholar's validated pipeline.

## Not in v1

Badge rewriting (PyPI/npm badges are already dynamic), git automation, batch repo scanning, monorepo support.

## License

MPL-2.0
