# PLAN.md

**Last updated:** 2026-10-14
**Status:** Exercise 5 — **step 2 (SPEC.md) complete.** Step 3 (plan, baseline,
trial run) is next. No modelling code yet.

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
- [x] **2026-10-14** — **Exercise 5, step 1** — looked at `data/leaves.csv`:
      518 rows, class counts (maple 216 / cherry 173 / walnut 129), scatter plot,
      and six anomalies including the three-block decimal-precision structure,
      26 duplicate rows and 3 contradictory labels.
- [x] **2026-10-14** — **Exercise 5, step 2** — wrote `SPEC.md` under the four
      lecture headings. Committed as draft (`0e70dbc`), then ran the interview
      and filled in the decisions. Within it: resolved the dataset provenance by
      finding the course's published copy and verifying a byte-identical hash
      (`b24ad636…f8a4e839`), and verified the split (104 test rows,
      `random_state=42`) and the grid sizes (68 candidates) by running them
      rather than estimating.

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
- [ ] **Solo or in pairs** for exercise 5? The assignment allows pairs; if so,
      `RUNS.md` needs a "who did what" section.
- [ ] Fate of the step-1 scratch scripts — 7 untracked `.py` files. See the
      "Still open" section at the end of `SPEC.md` for the recommendation.
      (`check_leaves3.py` in particular contains a duplicate-detection bug and
      should not be committed without a fix or a note.)

## Next step

**Exercise 5, step 3** — plan and baseline, per `SPEC.md`:

1. `uv init` and pin dependencies **first** (`AGENTS.md` §2 — a result from an
   unpinned environment is not reproducible):

   ```bash
   uv init
   uv add pandas scikit-learn scipy
   uv add --dev pytest
   ```

2. Then start step 3 with the assignment's own prompt:

   ```
   /plan Read SPEC.md and propose a plan
   ```

   Its first move is *"First list anything you would have to guess."*

3. Step 3 also requires the **baseline**: a model that always predicts the most
   common type. Measured expectation from `SPEC.md` — 41.35% (43/104) on the
   held-out rows. Any model must beat it.

## Conventions

- All Python through `uv`. Never global `pip install`.
- Run checks before calling anything done; record the command and its output.
- Keep this file current — it should never be more than one session stale.
- Append a row to `RUNS.md` after each substantial task.
