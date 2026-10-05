---
round: 1
focus:
  - "Cards 9 and 10 (`internal/shedadapters/bouncer.go`, `bouncerprompt.go`): does the Go guard on `CIRCLING` use the gating definition from the plan's Shared Decisions and the checkpoint from config, and is the cause `budget` whenever the `Bounces` seam reports `count >= budget`, even on a `CIRCLING` verdict?"
  - "Cards 4 and 11 (`internal/shedrecipe`, `internal/shedadapters`): do review segments take the budget from `loom.yaml`, and does the round after a budget `continue` spend no budget, with the `Bounces` seam and Shed agreeing on the count?"
  - "Cards 7, 8 and 10: are `round-<N>-escalation.md` and `round-<N>-parent-notice.md` written as the Shared Decisions specify, does a legacy decision file without `cause` read as `circling`, and do the circling verbs settle an escalation of either cause?"
  - "Every new refusal in the diff (cards 8, 10, 13): does it have a `refusal-spec.md` row and an untagged test that reaches it, per the plan's Shared Decisions?"
  - "Per-card mechanical check: for each card with `Rename` or `Delete` groups, did the group's type-specific check run against every one of the group's targets, notably card 15 (removal of the `recurring-finding` anomaly) and card 3 (config keys)?"
  - "Comment conventions on new or changed doc comments, especially `internal/shedadapters/doc.go`, `docs/overview.md` and the new test files: do they follow `CLAUDE.md` (semantic line breaks, no pinned tallies) and the surrounding Go style?"
---

Seed focus for round 1.
The range is every commit on the branch over the merge base with `main`; no rework round exists.
Cards 1, 2, 5, 6, 12, 14 and 16 are narrower and are left to the ordinary diff read.
