# PLAN.md

**Last updated:** 2026-10-14
**Status:** Scaffolding complete — repository set up, no code yet.

## Progress

### Done

- [x] **2026-10-14** — Installed `gh` 2.102.0 and `uv` 0.13.1 into
      `~/.local/bin` (user-scoped, no admin). Both verified working.
- [x] **2026-10-14** — Initialized git repo here on `main`, first commit
      `b0bdf63`.
- [x] **2026-10-14** — Created public repo
      [klaraseli/mlr-agent-lab](https://github.com/klaraseli/mlr-agent-lab)
      and pushed the initial commit.
- [x] **2026-10-14** — Added working rules (`AGENTS.md`, imported by
      `CLAUDE.md`) and the tracking files (`README.md`, `PLAN.md`, `RUNS.md`).

### Open questions

- [ ] Should course materials (lecture PDFs, test photos) live in a **separate
      private** repo? They are currently gitignored here. Note: 48 MB of phone
      JPEGs is a poor fit for plain Git — consider Git LFS if they are wanted
      under version control.
- [ ] Should commits use a different email than the currently configured
      `klarka.selingrova@gmail.com`? Decide before history builds up, since
      changing it later means rewriting commits.
- [ ] A live `ANTHROPIC_AUTH_TOKEN` is present in the shell environment. It is
      not in this repo and `.env` is now gitignored, but it should be rotated if
      it was ever exposed. See `AGENTS.md` §5.

## Next step

**Initialize the Python project and add the first real piece of work:**

```bash
uv init
uv add --dev pytest
uv run pytest
```

Then commit `pyproject.toml` and `uv.lock`, and replace this section with the
next concrete task.

## Conventions

- All Python through `uv`. Never global `pip install`.
- Run checks before calling anything done; record the command and its output.
- Keep this file current — it should never be more than one session stale.
- Append a row to `RUNS.md` after each substantial task.
