# AGENTS.md — Working Rules

Rules for any agent working in this repository. `CLAUDE.md` imports this file,
so these apply to Claude Code as well.

## 1. Read before you act

Before changing anything, read:

1. **`README.md`** — what this project is and how to run it.
2. **`PLAN.md`** — where we are and what the next step is.

If either file contradicts what the code actually does, say so *before* acting
on it. Do not silently trust a stale plan.

## 2. Python is always run through `uv`

This project manages Python with [uv](https://docs.astral.sh/uv/).

- **Run code:** `uv run <command>` (e.g. `uv run python script.py`,
  `uv run pytest`)
- **Add a dependency:** `uv add <package>`; dev-only: `uv add --dev <package>`
- **One-off tool:** `uvx <tool>`

**Never** install packages with a global `pip install`, `python -m pip install`,
or `pip install --user`. `uv` handles environments; global installs make runs
irreproducible and leak into other projects.

`pyproject.toml` and `uv.lock` are the source of truth. **Commit `uv.lock`** so
runs are reproducible.

## 3. Run checks before you claim it is done

A task is not done when the code is written. It is done when you have **run
checks and seen them pass.**

- Run the relevant checks (tests, linters, or at minimum actually executing the
  thing you changed).
- Report the **exact command** and its **observed result**.
- If a check **fails**, say so plainly and show the output. Do not describe a
  failing state as working.
- If you **could not** run a check, say that explicitly and explain why
  (e.g. needs interactive login, needs a network call). Never imply you verified
  something you did not verify.

Reporting "should work" or "looks correct" is not a substitute for running it.

## 4. Keep the tracking files current

- **`PLAN.md`** — update progress and the next step as work moves. It should
  never be more than one session stale.
- **`RUNS.md`** — append a row after each substantial task, recording the model
  used, what was asked, and how it was checked.

## 5. Never commit secrets

No API tokens, keys, passwords, or credentials — not in code, not in config, not
in commit messages.

- Keep secrets in environment variables or a local `.env` file.
- `.env` is gitignored. Verify that before adding files.
- If you **find** a secret in the repo, workspace, or logs: **flag it to the
  user and do not copy it anywhere** — not into a doc, not into a summary, not
  into `.gitignore` comments. Its value does not belong in any artifact.

## 6. Ask before anything outward-facing

Publishing is one-way. Before pushing, creating a public repo, or posting
anything externally:

- Show the user exactly what will be uploaded **first**.
- Get explicit approval.
- If asked for output from a file that does not exist or is empty, **stop and
  say so** rather than substituting different content.

---

## Definition of done

- [ ] `README.md` and `PLAN.md` were read
- [ ] Python was run via `uv`, with dependencies added via `uv add`
- [ ] Checks actually ran, and the command and result are reported
- [ ] `PLAN.md` reflects the new state; `RUNS.md` has a new row
- [ ] No secrets introduced; nothing outward-facing without approval
