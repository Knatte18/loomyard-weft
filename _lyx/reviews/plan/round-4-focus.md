---
round: 4
focus:
  - "In `_lyx/plan/11-tmux-tier-green.md`, settle whether the card's `**Edit:**` targets and its stop rule cover every file where its own stated cause site could be fixed: the card names `restartSession` and `addStrandIn`, which are test helpers in `internal/reedcli/smoke_lifecycle_test.go` and `internal/reedcli/smoke_test.go`, while the only target is `internal/reedcli/smoke_dotfill_test.go`, and reed production code under `internal/reedengine` is the intended stop condition."
  - "In `_lyx/plan/11-tmux-tier-green.md` and `_lyx/plan/12-publish-verify.md`, settle whether the plan guarantees that no card sets `publish_verify` while the `tmux` tier fails: check what card 11's stop branch does to the run, whether card 12 has a `Uses` edge on card 11, and whether card 12's `**Intent:**` states its precondition."
  - "In `_lyx/plan/09-mark-guards.md`, settle whether the `**Edit:**` list is stated as a candidate set under the card's own criterion (a test is marked only when it reads repository files outside its own package directory), so a listed file whose tests read only their own directory, or one such as `cmd/lyx/crosscompile_test.go` that builds through the compiler, may stay unchanged."
  - "In `_lyx/plan/11-tmux-tier-green.md`, settle whether every sentence of the `**Intent:**` bears on the implementer's branch choice, or whether it records a point-in-time measurement (machine, load, which commands passed) that goes stale."
---

Each entry names a plan location and the question to settle; none states an expected outcome.
