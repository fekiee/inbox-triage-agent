# CLAUDE.md — Project instructions

## Project
Inbox Triage Agent: a human-in-the-loop agent that identifies promotional email in an Outlook.com mailbox via Microsoft Graph and moves approved messages to Deleted Items. Full design: `docs/design.md`. Backlog: `docs/backlog.md`.

This is a **portfolio project** meant to reflect how an enterprise team would build it. Favour clarity, tests, and documented decisions over cleverness.

## Owner
Jin — RPA developer (UiPath) moving into agentic automation. Explain non-obvious engineering decisions briefly in PR descriptions; he will use them in interviews. Conversation may be in Spanish; **all code, identifiers, comments, commits, PRs and docs are in English**.

## Stack
- Python 3.12, managed with **uv** (`uv add`, `uv run`, `uv sync`)
- MSAL (device code flow) + Microsoft Graph REST via `httpx`
- SQLAlchemy + SQLite
- Pydantic for config and structured data
- Typer + Rich for the CLI
- Anthropic SDK (v2 only), behind a `Classifier` interface
- pytest, ruff (lint + format), mypy, pre-commit
- GitHub Actions for CI

## Layout
```
src/inbox_triage/
  auth.py        # MSAL, token cache
  graph.py       # Graph client: pagination, batching, backoff
  store.py       # SQLAlchemy models and repository
  rules.py       # rules engine (reads rules.yaml)
  classifier/    # Classifier interface + implementations (v2)
  policy.py      # decision policy
  review.py      # approval queue (CLI / CSV)
  executor.py    # move to Deleted Items, dry-run, undo
  cli.py         # Typer entry point
tests/
docs/            # design.md, backlog.md, adr/
rules.example.yaml
```

## Commands
- `uv sync` — install deps
- `uv run pytest` — tests
- `uv run ruff check . && uv run ruff format --check .` — lint
- `uv run mypy src` — types
- `uv run inbox-triage --help` — CLI

## Workflow
1. Every change starts from a GitHub issue.
2. Branch name: `feat/<issue-number>-short-name`, `fix/...`, `docs/...`, `chore/...`.
3. Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
4. Open a PR with `gh pr create`, link the issue (`Closes #N`), describe what/why/how-tested.
5. CI must pass. Jin reviews and merges — **never merge or push to `main` yourself**.
6. Significant design decisions get an ADR in `docs/adr/NNNN-title.md`.

## Safety rules (non-negotiable)
- **Never commit** `.env`, token caches, `*.db`, `exports/`, `logs/`, or any real email data. Check `git status` before committing.
- Tests use **synthetic fixtures only**; mock Graph — no network calls in tests.
- Destructive operations default to `--dry-run`. Executing requires an explicit `--execute` flag.
- Deletion = move to Deleted Items. Never use permanent delete endpoints.
- `never_touch` rules always override everything else.
- The LLM never triggers an action directly; it only produces a recommendation for the policy.
- Never log tokens, full email bodies, or previews of non-promotional mail.
- Do not run commands against the real mailbox unless Jin asks to in that session.
