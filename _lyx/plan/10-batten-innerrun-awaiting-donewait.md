# Card 10 — batten-innerrun-awaiting-donewait

**Edit:**
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#awaitingChildRemedy`
- `internal/battenshed#newInnerRunDeps`
- `internal/shedrecipe#innerRunEntry`
- `internal/shedrecipe#defaultInnerRunPollIntervalS`
- `internal/battenshed/innerrun_test.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/shedrecipe/entries_batten_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `contracts/recipes/batten-recipe.yaml`

**Prosa:**
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/battenshed#InnerRunDeps`
- `plan:internal/battenshed#ChildApproval`
- `internal/battenshed#haltedChildRemedy`
- `internal/battenshed#reportStuck`
- `internal/shedengine#OutputPointer`

**Intent:**
`Run-Shed` survives the PR hand-off and the driver's final report: it watches an `awaiting` child, resumes it once per approval, and finishes a `done` child only after its driver has ended or a grace window has elapsed.

`NewInnerRun` takes one more parameter, `driverExitGrace time.Duration`, stored on `innerRunProducer`, and resolves a nil `deps.Now` to `time.Now`.
`innerRunEntry` reads the optional int config key `driver_exit_grace_s` (default 900 when absent or zero, negative rejected naming the key, added to `configRejectUnknown`'s accepted keys) and passes it; declare the default as a constant beside `defaultInnerRunPollIntervalS` and update both doc comments.

New arms in `innerRunProducer.Call` (the running arm, the halted arms and the spawn logic are unchanged; `haltedChildRemedy` still covers `blocked`, `paused` and `failed`):

- **`awaiting`, no approval record** (`deps.ReadApproval` found false): sleep `pollInterval`, return Stuck with `BudgetExempt: true`, and a reason naming the child's awaiting reason (`status.Error`) and the hand-off: run `lyx loom approve` in the task worktree.
- **`awaiting`, approval not yet acted on**: the approval identity (`ApprovedAt` plus `HeadSHA`) differs from the one recorded in the marker `<scratchDir>/<name>-approval-acted`, or no marker exists.
  Call `deps.Spawn`, logging both Live-Substrate Spawn Observability lines around it as the existing spawn does.
  A Spawn error is today's hard error and writes no marker, so resuming the batten run retries the spawn.
  On success write the marker holding the identity and record the spawn confirmation (`recordSpawnConfirmed`), then sleep and return an exempt Stuck whose reason says the approved child was resumed.
- **`awaiting`, approval already acted on** (the marker matches): do not spawn; sleep and return an exempt Stuck whose reason says the approval at its `approved_at` was already acted on, and names the recovery: re-run `lyx loom approve` in the task worktree, which records a new identity and triggers another resume.
- **`done`**: on first sight write the marker `<scratchDir>/<name>-done-seen` holding `deps.Now()` in RFC 3339 (never overwritten while present).
  If `deps.DriverAlive` reports the strand gone, return Done.
  If it reports it live (a liveness error is logged at Warn and treated as live, so the wait stays bounded by the grace window) and less than `driverExitGrace` has passed since the marker's time, sleep and return an exempt Stuck with a reason naming the wait for the driver to finish its stop report.
  Once the grace has elapsed with the strand still live, log a warning naming the slug and the elapsed grace, and return Done: `Worktree-Teardown`'s session shutdown and `Remove`'s clean-worktree check take it from there.

An `ReadApproval` error, and a marker read or write failure other than "absent", are hard errors (mechanism failure), like a `ReadStatus` error.
Marker names are built by unexported helpers beside `SpawnConfirmedFile`; every Stuck reason goes through `reportStuck`.
`awaitingChildRemedy` is deleted as a hard-error text; the awaiting arm's hand-off wording is a new constant.
The running arm never sets `BudgetExempt`, so a running child keeps its 12-hour budget.

Rewrite `innerRunProducer.Call`'s disposition-table doc comment to describe the awaiting, resume and done-wait arms, and why the self-route's "sole Stuck arm" reasoning still holds (every new Stuck is a timed wait, not a stuck child).
In `innerrun_test.go`, `newInnerRunDeps` stubs `ReadApproval` (none), `DriverAlive` (not live) and a fake `Now`, and every `NewInnerRun` call passes a grace.

Tests, rewriting `innerrun_awaiting_test.go` and extending `innerrun_test.go`:
- awaiting without approval: sleeps once, returns exempt Stuck, reason names the awaiting reason and `lyx loom approve`, Spawn not called;
- awaiting with a new approval: spawns once, writes the marker, returns exempt Stuck; a second Call with the same approval and the child still awaiting does not spawn and its reason names re-running `lyx loom approve`; a Call after a new approval identity spawns again;
- awaiting with an approval whose Spawn errors: hard error, no marker, and the next Call spawns;
- `blocked`, `paused`, `failed` still hard errors carrying `haltedChildRemedy`;
- done with a live strand returns exempt Stuck until `DriverAlive` reports gone, then Done; done with the strand live past the grace (fake clock) returns Done;
- a done child's wait reaches Done through a full grace wait when driven through `shedengine` with the row's counted budget one short of exhausted (seed the history with N-1 counted Run-Shed Stucks and a `MaxBounces` of N).
`entries_batten_test.go`: `driver_exit_grace_s` default, explicit value, and negative rejection.

Recipe and docs:
- `batten-recipe.yaml`: add `driver_exit_grace_s: 900` to Run-Shed's config and rewrite the header comment: the 12-hour budget counts the running watch only; the awaiting wait and the driver-exit wait are budget-exempt bounces, the latter bounded by the grace window; row names and `max_bounces: 1440` unchanged.
  Mirror the new key in `lifecycle_integration_test.go`'s `shortPollBattenRecipe` copy.
- `internal/battenshed/doc.go`: the paragraph saying teardown follows the child's terminal state and not the driver's exit now says the done arm waits for the driver strand up to the grace window, and that a driver still alive past it is ended by teardown.
- `docs/overview.md`'s batten section: replace the awaiting sentence (approve, resume the child with `lyx loom start`, resume batten) with the new flow: batten watches the awaiting child, the operator runs `lyx loom approve` in the task worktree, batten resumes the child itself once per approval, and waits for the driver before teardown.

**ImpactSummary:** Changes Run-Shed's awaiting and done dispositions and NewInnerRun's signature (one registry caller); running and halted-child behavior is unchanged.
