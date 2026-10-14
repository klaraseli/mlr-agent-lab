# mlr-agent-lab

## Goal

A lab notebook for working with coding agents on MLR (7th semester)
coursework. The point is not just to produce answers, but to keep a record of
**how** each result was produced and **how it was checked**, so results stay
trustworthy and reproducible.

## How to run things

This project uses [uv](https://docs.astral.sh/uv/) for Python. No global
`pip install` — see `AGENTS.md`.

Current setup: Python 3.12.1, no `pyproject.toml` yet. Initialize it on first
use:

```bash
uv init                  # creates pyproject.toml
uv add <package>         # add a runtime dependency
uv add --dev <package>   # add a dev-only dependency
```

Run anything inside the managed environment:

```bash
uv run python            # interactive Python
uv run python script.py  # run a script
uv run pytest            # run tests
uvx <tool>               # run a one-off tool without installing it
```

`uv.lock` is committed so runs are reproducible.

## Repository layout

| File | Purpose |
|---|---|
| `README.md` | This file — goal and how to run things |
| `AGENTS.md` | **Working rules for agents.** Read first. |
| `CLAUDE.md` | Imports `AGENTS.md` so Claude Code loads those rules |
| `SPEC.md` | The task: goal, context, constraints, done-when. Read second. |
| `PLAN.md` | Progress and the next step |
| `RUNS.md` | Log of agent sessions: model, request, how it was verified |
| `data/leaves.csv` | The dataset — 518 leaves, three tree types. Origin and hash in `SPEC.md`. |

## Working rules

Read `AGENTS.md` before making changes. In short: read `README.md` and
`PLAN.md` first, use `uv` for all Python, run checks before claiming anything is
done, and never commit secrets.

## Notes

Local course materials (lecture slides, test photos, cheat sheets) are
intentionally **not** committed here — see `.gitignore`.
