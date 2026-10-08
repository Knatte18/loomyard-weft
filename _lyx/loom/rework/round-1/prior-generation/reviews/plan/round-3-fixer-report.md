# Plan-Review round 3 — fixer report

All three findings were checked against the code and fixed.
`lyx loom validate-plan` exits clean (informational `create-new-unit` only), and `lyx quarry resolve` finds the new `internal/battenshed#innerRunProducer.awaitingStep` glyph.

## Fixed

- **F1** (LOW, decision): confirmed. `circlingWayMalformed` (`internal/loomcli/circling.go:40`) names `lyx loom start`, and `awaitingStep`'s decide-again hint (`internal/battenshed/innerrun.go:764`) names approve or reject only.
  - `00-overview.md`, Shared Decision `halted-paths-keep-start`: `circlingWayMalformed` and its refusal-spec row `circling: escalation record malformed` are now a stated exception that keeps `lyx loom start`, with the reason (the fix needs the round re-escalated, and `resume` with no pending decision refuses).
  - `09-resume-circling.md`: `internal/battenshed#innerRunProducer.awaitingStep` joins the `Edit` group; the batten bullet makes the decide-again hint name the PR-Gate and review-segment hand-offs, as `awaitingHandOff` does, and gives its reachability through a `goto` from an approved PR-Gate; the bound now says batten's two awaiting hint texts are its only change; the `innerrun_awaiting_test.go` test bullet extends the existing decide-again assertions to expect `lyx loom resume`, with no new test.
- **F2** (LOW, consistency): confirmed. `08-fix-start.md` now declares `plan:internal/burlerengine#Engine.decideRound` with its declaration head in `Create`, names it in the Intent, and states what it takes and how it handles a fixer `after-review` never started.
- **F3** (LOW, consistency): confirmed against `docs/overview.md:387-388` and `resume.go`'s header. `09-resume-circling.md` now edits both: the overview's `lyx loom resume` sentence and the `resume.go` file header name the awaiting-at-a-Bouncer-row case with a pending circling decision beside the halted case.
  `resume.go` is not listed as a file target, since the card already names members in it and a file target beside them is `redundant-file-target`.

## Disputed

None.

## Deferred

None.
