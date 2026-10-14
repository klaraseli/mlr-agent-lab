# RUNS.md

A log of agent sessions on this repository: which model ran, what was asked, and
how the result was verified.

Add a row per substantial task. The **verification** column should name the
actual command or check, so the claim can be re-tested later.

## How to find the model

Run **`/status`** in Claude Code. It reports the active model and where the
configuration comes from.

## Environment

As of **2026-10-14**, from `/status` and the environment:

| Setting | Value | Source |
|---|---|---|
| `/model` default (saved) | `thinker` | user `settings.json` |
| `ANTHROPIC_MODEL` | `agentic` | environment — **overrides** the saved default |
| Opus-class alias | `thinker` | `ANTHROPIC_DEFAULT_OPUS_MODEL` |
| Haiku-class alias | `mini` | `ANTHROPIC_DEFAULT_HAIKU_MODEL` |
| Effort | `high` | `CLAUDE_EFFORT` |
| API endpoint | `https://llm.ai.e-infra.cz/` | `ANTHROPIC_BASE_URL` |
| Claude Code | 2.1.299 | `claude --version` |

**Caveat on model naming:** `thinker`, `agentic`, and `mini` are aliases served
by the endpoint above (a non-Anthropic gateway), not official Claude model IDs.
They are recorded here as they appear in `/status` and the environment. When
reporting which model ran a task, quote `/status` verbatim rather than
inferring a model family from the alias name.

> **Note:** this file records configuration, not secrets. Never paste
> `ANTHROPIC_AUTH_TOKEN` or any other credential here. See `AGENTS.md` §5.

## Log

| Date | Model | What I asked | How it was checked | Result |
|---|---|---|---|---|
| 2026-10-14 | `/model` → `thinker`; `ANTHROPIC_MODEL=agentic` active | Install `gh` and `uv` without admin rights, into my user folder, and verify both work | `gh --version`, `uv --version`, `uvx --version`; `uv run python -c ...` printed 3.12.1; `uvx ruff --version` ran a real package; `where` from native `cmd.exe`; uv SHA-256 matched publisher checksum | Passed. gh 2.102.0, uv 0.13.1 in `~/.local/bin` (already on user PATH). gh needed interactive `gh auth login` — reported, not claimed as done. |
| 2026-10-14 | `/model` → `thinker`; `ANTHROPIC_MODEL=agentic` active | Use `gh` to create a public repo `mlr-agent-lab` from this folder; show what will be uploaded before pushing | **Stopped:** target folder was empty (verified with `find`, `du`, native `dir /a`). After user chose starter files: staged diff reviewed, secret/PII scan clean, remote tree re-read via `gh api .../git/trees/main` | Passed. Repo is public with 2 files (`.gitignore`, `README.md`), commit `b0bdf63`. Diverged from the literal request — folder was empty, so content came from the user's follow-up choice, not from pre-existing files. |
| 2026-10-14 | `/model` → `thinker`; `ANTHROPIC_MODEL=agentic` active | Write `AGENTS.md` (rules), have `CLAUDE.md` load it, and create `README.md`, `PLAN.md`, `RUNS.md` | See below | Passed — 5 docs created, `@AGENTS.md` import confirmed working in a fresh session |

### 2026-10-14 — setup and docs session

- **Model:** confirmed from `env` and `~/.claude/settings.json`, not guessed.
  Nothing was inferred from the alias names.
- **Checks run:**
  1. All five files verified present on disk with byte counts.
  2. `@AGENTS.md` import **tested for real** — a fresh `claude -p` session, told
     *not* to read any file, correctly recited the `uv` rule ("run code via
     `uv run`, add packages with `uv add`, never a global `pip install`"). This
     confirms `CLAUDE.md` → `AGENTS.md` loading works.
  3. Secret scan across all docs for real token *values*
     (`sk-`/`gho_`/`ghp_` + 16 or more chars): no matches. One earlier hit was
     this file's own descriptive prose, not a credential.
  4. `.env` confirmed ignored via `git check-ignore` (was **not** ignored
     before this session; `.gitignore` was corrected).
- **Config note:** the fresh session reported
  `claude-code:unrecognized_model {"model":"agentic"}` — Claude Code does not
  recognize `agentic` and assumes a 200k context window. Harmless for these
  tasks, but relevant if long-context work is planned against this gateway.
