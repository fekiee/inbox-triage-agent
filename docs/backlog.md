# Backlog

Each item becomes a GitHub issue. Labels: `v1`, `v2`, `v3`, plus `infra`, `feature`, `docs`, `test`.

## Milestone v1 — Rules + human-in-the-loop (no LLM)

1. **Project scaffolding** `infra`
   As a developer, I want a uv-managed Python package with ruff, mypy, pytest and pre-commit so the codebase has quality gates from day one.
   *AC:* `uv run pytest` and lint pass on an empty package; `.gitignore` extended (`.env`, token cache, `*.db`, `exports/`, `logs/`).

2. **CI pipeline** `infra`
   As a reviewer, I want GitHub Actions to run lint, types and tests on every PR.
   *AC:* workflow on `pull_request`; branch protection on `main` documented in README.

3. **Entra ID app registration guide** `docs`
   As a user, I want step-by-step docs to register the app and grant `Mail.ReadWrite`.
   *AC:* `docs/setup-entra-id.md`; `.env.example` with required variables.

4. **Authentication (device code flow)** `feature`
   As a user, I want to log in once and have the token cached securely.
   *AC:* `inbox-triage login`; token cache in ignored path; silent refresh.

5. **Graph client** `feature`
   As the system, I need to page through the inbox fetching metadata only, with batching and 429 backoff.
   *AC:* unit tests with mocked responses for pagination and throttling.

6. **State store** `feature`
   As the system, I persist messages, senders, decisions, actions and runs in SQLite.
   *AC:* models per design §5; fetch is idempotent/resumable.

7. **Rules engine** `feature`
   As a user, I define `never_touch`, `auto_delete` and heuristics in `rules.yaml`.
   *AC:* allowlist always wins; `List-Unsubscribe` heuristic; `rules.example.yaml` committed, real `rules.yaml` ignored.

8. **Approval queue (CLI + CSV)** `feature`
   As a user, I review candidates grouped by sender and approve/reject in bulk.
   *AC:* `inbox-triage review`; table with sender, count, sample subjects, reason; CSV export/import.

9. **Executor with dry-run and undo** `feature`
   As a user, I want approved messages moved to Deleted Items, with a dry-run default and the ability to undo a run.
   *AC:* `--execute` required; `inbox-triage undo <run_id>`; every action in audit log.

10. **Feedback loop** `feature`
    As a user, my approvals/rejections update the rules so I am not asked again.

11. **Run report** `feature`
    As a user, I see a summary after each run (fetched, by rules, queued, moved).

12. **README v1** `docs`
    Problem, architecture diagram, setup, usage, safety model, screenshots (synthetic data).

## Milestone v2 — LLM classifier + evaluation

13. ADR: choice of LLM provider and model.
14. `Classifier` interface + Claude implementation with structured outputs, classifying per sender.
15. Decision policy with confidence thresholds.
16. Cost tracking and per-run budget limit; Batch API for bulk classification.
17. Golden dataset tooling (label senders locally, never committed).
18. Eval harness: precision / recall / FPR for rules vs LLM vs combined.
19. Optional: Ollama implementation and comparison in eval.

## Milestone v3 — UX + automation

20. Web approval UI (FastAPI or Streamlit).
21. ADR: scheduled execution host.
22. Scheduled runs + notification with run report.
23. Optional: one-click unsubscribe (RFC 8058) after security review.
