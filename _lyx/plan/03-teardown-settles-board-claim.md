# Card 3 — teardown-settles-board-claim

**Create:**
- `internal/pairteardown#claimSettlement`
- `internal/pairteardown#claimKeep`
- `internal/pairteardown#claimDone`
- `internal/pairteardown#claimClear`
- `internal/pairteardown#claimHeld`
- `internal/pairteardown#settleClaim`
- `internal/pairteardown#Teardown.childLanded`
- `internal/pairteardown#Teardown.battenRunState`
- `internal/pairteardown#Teardown.readBoardClaim`
- `internal/pairteardown#Teardown.writeBoardClaim`
- `internal/pairteardown#Teardown.settleBoardClaim`

**Edit:**
- `internal/pairteardown#Request`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown/doc.go`
- `internal/pairteardown/teardown_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/battencli#teardownRequest`
- `internal/battencli/lifecycle_integration_test.go`
- `pattern/PATTERN-pair-teardown.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D10: every teardown route releases the board entry's run claim.
`Request` gains `Landed bool`, which batten's `teardownRequest` sets, since its row runs only after a done child.
`Teardown` gains four substrate seams beside its existing ones, wired by `New`: `landed` to `childLanded`, `battenRun` to `battenRunState`, `readClaim` to `readBoardClaim` and `writeClaim` to `writeBoardClaim`.
`RemovePair` asks the existing `gone` substrate whether the task worktree is gone and, while it is present, reads `landed` before the removal;
a gone task worktree is not landed with no read and no error, since its status file went with it, so the settle falls through to the batten-state rule.
`RemovePair` ORs that answer with `req.Landed`, and after a successful removal calls `settleBoardClaim`;
the settle only warns on a failure, and the removal result is returned unchanged.
`childLanded` reads the task worktree's own status through `state.ReadJSONStrict` over `shedrun.StatusFile` and `shedrun.StatusLock`, and reports a history entry naming `loomshed.NameFinalize` with outcome Done.
`settleClaim` is the pure rule: a nil status, `done`, or any status without the run form (`boardengine.IsRunStatus`) is `claimKeep`;
landed is `claimDone`;
otherwise `battenSeeded` false, or a batten state among `paused`, `blocked`, `failed` and `awaiting`, is `claimClear`;
batten state `done` is `claimDone`;
batten state `running` is `claimHeld`.
`readBoardClaim` and `writeBoardClaim` open the board through `boardengine.OpenHub` over the prime's hub path and read the entry's status or write it through `Board.SetStatus`;
`battenRunState` reads batten's seed presence through `shedrun.ReadSeed` on the prime and its state from the prime's run status file for the slug.
`settleBoardClaim` reads the claim and, unless `landed` already decides it, the batten run through the seams, applies `settleClaim`, and writes: `claimDone` sets the status `done`, `claimClear` clears it with a nil status, `claimHeld` warns naming `lyx batten status <slug>` and `lyx batten pause <slug>` as the step that turns a running run into an abandon, and `claimKeep` writes nothing.
A board or status read failure warns and writes nothing.
What the clearing lets through: scope edits on an entry whose batten run is halted or that batten never seeded, since `board-lock-running` no longer refuses them, and one landed case, a run whose Finalize merged, failed its done mark and returned Stuck on its push, so its history holds no `Finalize` Done and its entry ends cleared, not done, for the operator to re-mark with `lyx board set-status`.
What bounds it: `claimClear` never fires while batten's run for the slug is `running`, never when the child history holds a `Finalize` Done or the request is landed, and only after the session has ended and both worktrees are removed, so no step of the run can still write.
Tests: `TestTeardown_CallSequence` drives the rule through `Run` with fake `landed`, `battenRun`, `readClaim` and `writeClaim` substrates, one row per rule outcome (absent, done and non-run-form entries kept, landed from the request and from the child read marked done, unseeded and each halted batten state cleared, batten `done` marked done, batten `running` held with no write, a claim read failure writing nothing, and a gone task worktree over a run-form entry with no batten seed cleared with no `landed` read), each row asserting the call order and the written status, and a failed removal row asserting no claim substrate runs;
`TestRun_TeardownScenario` gains steps over hubforge pairs with a run-form board entry, driving the real `childLanded`, `battenRunState`, `readBoardClaim` and `writeBoardClaim`: with no batten seed the entry is cleared after `Run`, with a child status whose history holds a `Finalize` Done it is marked done, and with the prime's batten status `running` it is left unchanged;
batten's four-row lifecycle step asserts the board entry reads `done` after its `Worktree-Teardown` row, which only the request's landed flag decides there, since the stubbed child status holds no `Finalize` Done and batten's own run is still `running`.
The package doc and the pair-teardown background gain the claim clause.

**ImpactSummary:** `pairteardown.Request` gains a field and `RemovePair` gains a post-removal board write; `lyx fabric remove` and batten's teardown row both reach it through the composite, and no caller's signature changes.
