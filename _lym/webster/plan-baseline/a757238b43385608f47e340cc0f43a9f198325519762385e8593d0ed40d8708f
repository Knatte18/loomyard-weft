---
format: 5
approved: true
language: go
---

# Plan: orch full autonomy mode

Add `autonomy`, a per-task beam.yaml setting found by dobbyengine's one lookup from a run's or an orch's own worktree upward.
Under it, the run's parent orch settles every halt, and the operator is reached only for a severe halt or a merge-ready beam.
The plan adds the key and its reader, stamps each halt notice with the run's value, forces the parent asks on and the operator seats off,
lets the parent's recorded satisfied answer end a member's PR-Review, posts `[beam drained]` from Beam-Await,
filters the Slack reply hook to operator-marked replies, and tells the orch its autonomy at launch and in a new on-demand role file.

## Card Index

1 — beam-autonomy-key — add beam.yaml's per-task `autonomy` key and LookupAutonomy
2 — finalize-notice-skips-autonomy — leave `beam.autonomy` out of Finalize's config-change notice
3 — hubgeom-autonomy-reader — one spawn-free reader of a run's or an orch's autonomy, false with a warning on failure
4 — halt-notices-carry-autonomy — stamp every halt notice of a run with `autonomy: <true|false>`
5 — parent-ask-clear — let a run's ended parent ask be cleared so its seat asks again
6 — drift-autonomy — force the judge's parent ask on and interactivity off, and halt on an unanswered judge ask
7 — loom-autonomy-forcing — force the PR-Review ask, Discussion and raddle keys, and add `pr_review_parent_timeout_min`
8 — pr-record-autonomy — record the review's autonomy in the pull request record and clear it with the session
9 — pr-review-ask-helpers — one PR-Review asker and one standing reader over the run's ask record
10 — pr-review-session-autonomy — record autonomy once per review, start the session non-interactive and ask the parent
11 — pr-review-ending-autonomy — end the review only on a satisfied answer at the gated head; halt on a closed pull request
12 — parent-review-ask-verb — add `lym loom parent-review ask` and the PR-Review answer's way forward
13 — pr-continue-autonomy — gate `pr-continue` on the parent's satisfied answer at the current head
14 — goto-clears-pr-review — clear the record's autonomy and session on a goto onto PR-Review
15 — beam-await-drained — post one `[beam drained]` line when an autonomous beam's work is done
16 — slack-hook-operator-filter — forward only `[operator]` replies to Slack under autonomy
17 — orch-role-autonomy-texts — add `role-autonomy.md`, its exceptions in the role and help texts, and two glossary lines
18 — orch-pointer-autonomy-line — append the orch's autonomy line to every pointer beside the presence step

## Shared Decisions

### Decision: one reader, read at call time

- **Decision:** every consumer reads autonomy through `hubgeom.Autonomy` (card 3) at the moment it acts, never from a value captured at wiring.
  The one exception is PR-Review, which records the run's value in the pull request record once per review (cards 8 and 10).
- **Rationale:** D2 and D9: a hand edit, a reconcile or a pulled default layer takes effect at the next halt, wake or turn.
- **Applies to:** cards 4, 6, 7, 10, 15, 16 and 18

### Decision: forcing on the resolved config

- **Decision:** loom's and drift's `ApplyAutonomy` force the keys on the resolved config, and the run's setting lookup answers the same forced values through `AutonomyForcedSetting`; no layer file is written.
- **Rationale:** D3: one value per key wherever it is read, and autonomy off restores each layered value at the next load.
- **Applies to:** cards 6 and 7
