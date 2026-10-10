# Card 3 — teardown-settles-board-claim

**Create:**
- `plan:internal/pairteardown#claimSettlement` -> `type claimSettlement string`
- `plan:internal/pairteardown#claimKeep` -> `const claimKeep claimSettlement = "keep"`
- `plan:internal/pairteardown#claimDone` -> `const claimDone claimSettlement = "done"`
- `plan:internal/pairteardown#claimClear` -> `const claimClear claimSettlement = "clear"`
- `plan:internal/pairteardown#claimHeld` -> `const claimHeld claimSettlement = "held"`
- `plan:internal/pairteardown#settleClaim` -> `func settleClaim(status *string, landed, battenSeeded bool, battenState string) claimSettlement`
- `plan:internal/pairteardown#Teardown.childLanded` -> `func (t *Teardown) childLanded(slug string) (bool, error)`
- `plan:internal/pairteardown#Teardown.settleBoardClaim` -> `func (t *Teardown) settleBoardClaim(slug string, landed bool) error`

**Edit:**
- `internal/pairteardown#Request`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown/doc.go`
- `internal/pairteardown/teardown_test.go`
- `internal/battencli#teardownRequest`
- `internal/battencli/wire_test.go`
- `pattern/PATTERN-pair-teardown.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D10: every teardown route releases the board entry's run claim.
`Request` gains `Landed bool`, which batten's `teardownRequest` sets, since its row runs only after a done child.
`Teardown` gains two seams, `landed func(slug string) (bool, error)` and `settle func(slug string, landed bool) error`, wired by `New` to `childLanded` and `settleBoardClaim`.
`RemovePair` reads `landed` before the removal while the task worktree is present, ORs it with `req.Landed`, and after a successful removal calls `settle`;
a settle failure only warns, and the removal result is returned unchanged.
`childLanded` reads the task worktree's own status through `state.ReadJSONStrict` over `shedrun.StatusFile` and `shedrun.StatusLock`, and reports a history entry naming `loomshed.NameFinalize` with outcome Done.
`settleClaim` is the pure rule: a nil status, `done`, or any status without the run form (`boardengine.IsRunStatus`) is `claimKeep`;
landed is `claimDone`;
otherwise `battenSeeded` false, or a batten state among `paused`, `blocked`, `failed` and `awaiting`, is `claimClear`;
batten state `done` is `claimDone`;
batten state `running` is `claimHeld`.
`settleBoardClaim` opens the board through `boardengine.OpenHub` over the prime's hub path, reads the entry, reads batten's seed presence through `shedrun.ReadSeed` on the prime and its state from the prime's run status file for the slug, applies `settleClaim`, and writes: `claimDone` sets the status `done` through `Board.SetStatus`, `claimClear` clears it with a nil status, `claimHeld` warns naming `lyx batten status <slug>` and `lyx batten pause <slug>` as the step that turns a running run into an abandon, and `claimKeep` writes nothing.
A board or status read failure warns and writes nothing.
Tests: a new table test in `teardown_test.go` over `settleClaim` covers every row of the rule, and `TestTeardown_CallSequence` gains steps asserting the settle seam runs after a successful removal with the landed flag from the request or the child read, and never after a failed removal;
`wire_test.go` gains a test asserting `teardownRequest` marks the removal landed.
The package doc and the pair-teardown background gain the claim clause.

**ImpactSummary:** `pairteardown.Request` gains a field and `RemovePair` gains a post-removal board write; `lyx fabric remove` and batten's teardown row both reach it through the composite, and no caller's signature changes.
