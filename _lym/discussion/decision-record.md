## Goal

```markdown
## Seen (orch, 2026-11-17)

No lym commit path refuses a secret file: `record-batch` and fabric's commit verbs commit whatever the agent staged.
Unattended Codex and Claude sessions now commit on their own.
Source: the survey F17 (`orchestrator-survey-ideas`), from ccswarm, which blocks `.env`, `*.pem`, `id_rsa`, `credentials*` and `secrets.y*ml`.

## Idea

- A Go check in `record-batch` and fabric's commit path refuses a commit that adds a path matching a deny-list of secret-file patterns.
- The list ships with lym, and a hub may add patterns in its config.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - added-files-net-tree-delta (class design, severity BLOCKING, open in rounds 1)
  - correctness-finding-hides-secret-check (class design, severity MEDIUM, open in rounds 1)
  - hub-patterns-untested-through-wiring (class design, severity MEDIUM, open in rounds 1)
  - master-recover-batch-keeps-secret (class design, severity BLOCKING, open in rounds 1)
  - new-tests-skip-t-parallel (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
