# Card 11 — batten-halted-wait

**Edit:**
- `internal/battenshed/innerrun.go`
- `internal/battenshed/doc.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `contracts/recipes/batten-recipe.yaml`

**Create:**
- `internal/battenshed/innerrun_halted_test.go`

**Prosa:**
- `docs/overview.md`
- `CLAUDE.md`

**Intent:**
Batten waits through a halted child instead of exiting `failed`, so the pair keeps the watcher that lands and tears it down (Decision 6, fixes #338).

In `innerRunProducer.Call`, the `blocked`, `paused` and `failed` arm stops returning a hard error and returns a budget-exempt wait through `exemptWait`, like the `awaiting` arm.
The stuck reason names the child's state, its `Error` and `CurrentProducer`, and the resume command: `lyx loom start` in the task worktree; this run then keeps watching.
Batten never spawns or resumes a halted child itself.
When the child is `running` again the running arm takes over, and `done` reaches teardown as today.

Batten logs one `logger.Warn` per halt episode: a marker file in the producer's scratch directory, beside the existing markers and named the same way, records the child's history length at the last Warn, and the arm Warns only when the marker is absent or records a different length.
A resume and re-halt grows the history, so it Warns again; the polls between stay quiet.
A marker read or write failure is a hard error, as the other markers' are.

Bound: the wait has no time limit and spends no bounce budget, so a child nobody resumes keeps batten polling every `poll_interval_s`; `lyx batten pause <slug>` stops the wait and `lyx batten status` shows the reason.
`failed` is included because `lyx loom start` resumes it like `blocked` and `paused`.

Replace `haltedChildRemedy` with the new reason text, and rewrite `Call`'s disposition table and the self-route paragraph after it, which say a halted child is always a hard error.
Update `battenshed/doc.go` where it describes what Run-Shed watches, the header comment of `contracts/recipes/batten-recipe.yaml` (a halted child is now a budget-exempt wait, and Run-Shed's own hard-error return is what still halts the run `failed`), and `docs/overview.md`'s batten paragraph, which says a halting child fails `Run-Shed`.
Delete `CLAUDE.md`'s TEMPORARY item for #338.

Tests: in `innerrun_halted_test.go`, each of `paused`, `blocked` and `failed` returns a budget-exempt Stuck whose reason carries the state, the error, the current producer and `lyx loom start`, with no spawn; many polls at one history length Warn once, and a longer history Warns again (capture the Warn through the logger the package's other tests use, or assert the marker's content if no capture exists); a `paused` child followed by `running` then `done` reaches the done arm.
Remove `TestInnerRun_OtherHaltedStatesKeepHaltedChildRemedy` from `innerrun_awaiting_test.go`.
In `internal/battencli/lifecycle_integration_test.go`, replace `TestBattenIntegration_RunShedBlocked_LeavesThePairIntact` with a run whose read-status answers `paused`, then `running`, then `done`: the run survives the paused polls with the pair intact, and reaches Worktree-Teardown, which removes the pair.

**ImpactSummary:** Turns Run-Shed's halted-child hard error into a budget-exempt wait; the running, awaiting and done arms are unchanged.
