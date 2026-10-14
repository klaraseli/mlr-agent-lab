# CLAUDE.md

This repository keeps its working rules in a tool-neutral file so any agent can
use them. Claude Code loads them via the import below.

@AGENTS.md

## Notes for Claude Code

Everything in `AGENTS.md` applies. Two reminders that matter most in practice:

- **`uv` only.** No global `pip install`. Use `uv run` / `uv add` / `uvx`.
- **Run checks, then report the exact command and its real output.** Do not
  claim something works without having executed it.
