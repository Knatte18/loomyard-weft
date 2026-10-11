# Card 16 — batten status flags a halted batten over a done child

**Create:**
- `plan:internal/battencli#haltedOverDoneChild` -> `func haltedOverDoneChild(slug string, st, child shedengine.Status, childFound bool) (string, bool)`
- `plan:internal/battencli#battenCLI.readChildStatus` -> `func (c *battenCLI) readChildStatus() (shedengine.Status, bool)`
- `plan:internal/battencli#childDoneFlagKey` -> `const childDoneFlagKey = "child_done_batten_halted"`

**Edit:**
- `internal/battencli#battenCLI.battenStatusExtras`
- `internal/battencli/run_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/battencli#taskWorktreeLocation`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`

**Intent:**
Flag a batten run that stopped before its teardown while its child loom run is done (decision D10), so `lyx batten status <slug>` shows the stall and the step that closes it.

`haltedOverDoneChild` is pure: it answers the flag text and true when batten's `st.State` is failed, blocked or paused (not running, not done, assumption A5) and `childFound` holds with `child.State` done; the text reads `batten is <state> while its child loom run is done; run "lyx batten run <slug>" from the prime, which runs Worktree-Teardown`.
`readChildStatus` resolves the task worktree with `taskWorktreeLocation(c.location, c.slug)` and reads `shedrun.StatusFile(task, shedrun.SelfRunID)` through its status lock, as the Run-Shed wiring's `ReadStatus` does; an absent pair, an absent or unreadable file and a lock that cannot be taken all answer found false, since failing to read the child must never change batten's own status answer.
`battenStatusExtras` adds `childDoneFlagKey` with the flag text when `haltedOverDoneChild` answers true, and no key otherwise, including when the pair or the child status is gone.

Test: a new table test in `run_test.go` over `haltedOverDoneChild`: failed, blocked and paused batten over a done child carry the flag naming `lyx batten run <slug>`; a running or done batten, a child that is not done and an absent child carry none.
Raise `internal/battencli`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.

**ImpactSummary:** One status key added to batten's status envelope; the generic status body and every other key are unchanged.
