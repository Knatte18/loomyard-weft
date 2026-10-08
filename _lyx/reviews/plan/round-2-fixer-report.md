# Plan-Review round 2 — fixer report

Each finding was checked against the code and the decision record.
F1, F3 and F4 held and are fixed; F2's premise is false, so it is disputed.
`lyx loom validate-plan` passes on the fixed plan, with informational `create-new-unit` findings only, so every new glyph and `plan:` handle resolves or parses.

## Fixed

- **F1** (cards 1, 2, 3, 5, 6, 7, 8, 9): each `Edit` group lists member glyphs, resolved through `lyx quarry toc`, in place of non-test Go file paths.
  Test files, YAML, markdown and the `doc.go` package docs stay file paths.
  Each new symbol in an existing file is now a `Create` handle with its declaration head: `loomengine#DiscussionDecisionRecordRel` and `loomcli#writeCarryOver` (card 3), `burlerengine#FixStart`, `FixStartParallel` and `FixStartAfterReview` (card 8), and `shedadapters#PendingCirclingDecision` (card 9).
  No re-sign arrow is needed, since no listed member's signature changes; cards 2, 5, 7, 8 and 9 now say so.
  Card 8 states that `join` keeps its signature and serves the parallel order, while the after-review order waits on the reviewer in `Run`, and the failure rules both orders share move into one unexported helper in `handoff.go`.
  Card 9 names where the resume verb's Bouncer-subdir lookup lives: a new `loomCLI` field `bouncerSubdir`, which `wire` fills with `loomrecipe.BouncerRunSubdir`.
  The file headers each card edits (`append.go`, `config.go`, the two `circling.go` files) are described as the headers of the files that declare the listed members.
- **F3** (card 9): `PendingCirclingDecision` resolves the round through `ResolveRound`, as `RecordCirclingDecision` does.
  Every error it returns names the path it concerns; the decision file's parse errors, which `readCirclingDecision` returns without the path, are wrapped with it.
  The refusal's way forward now fits every cause: fix the file or directory the message names, and for a malformed decision file delete it and record the decision again with the circling verbs, then re-run `lyx loom resume`.
- **F4** (card 9): `internal/battenshed#awaitingHandOff` and `innerrun_awaiting_test.go` join the Edit list.
  The hint names both hand-offs: approve or reject at PR-Gate, or the circling verbs and then `lyx loom resume` at a review segment's escalation.
  `ReadDecision` and batten's resume of the child are unchanged.
  The test drops its assertion that the hint never names `lyx loom resume`, together with its doc premise.
  The `docs/overview.md` bullet also covers the batten paragraph on an `awaiting` child (`docs/overview.md:477`), which names only approve and reject.

## Disputed

- **F2**: the premise that a settled accept can sit under an `awaiting` status is false.
  `shedengine` persists `StateRunning` before it calls any producer when the run was halted (`internal/shedengine/run.go:225-229`, "the resume write").
  `settleCirclingAccept` runs only inside `Bouncer.Call` (`internal/shedadapters/bouncer.go`, `settleUnconverged`), so the decision is settled only after the status has left `awaiting`.
  A crash after the settle write therefore leaves the run `running`, which `resumeFromRunState` already handles.
  A failed Approve or Commit returns an error, which records the run as failed, not awaiting.
  The Bouncer returns `Awaiting` only from `escalate`, which `settleUndecided` alone reaches, so the round has no decision file at that point.
  To keep the point from being raised again, card 9 now states this fact in its `PendingCirclingDecision` bullet.

## Deferred

None.
