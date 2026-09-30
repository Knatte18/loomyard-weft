# Card 3 — start picks the status strand by the recorded driver

**Edit:**
- `internal/loomcli/start.go`
- `internal/loomcli/sharedbootstrap.go`
- `internal/loomcli/start_watchdog_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `CONSTRAINTS.md`
- `docs/overview.md`

**Uses:**
- `internal/loomcli#removeStatusStrands`
- `internal/loomcli#seedGoDriverRun`
- `internal/loomcli#seedLLMDriver`
- `internal/loomcli#statusStrandDisplayName`

**Intent:**
Make `start`'s step 4 choose the session's driving surface from the driver `seedAndCommitBootstrap` already returns, so an llm-driven run shows the ly-drive strand and never the `loom-status` strand, while a go-driven run keeps today's status strand.
No second driver read: the value already in hand in `startCmd`'s `RunE` is the branch condition.

In `sharedbootstrap.go`, drop the `c.reed.Up()` call from `ensureStatusStrand`; the rest of its keep/replace/add logic and the sidecar write stay byte-for-byte.
Its doc comment now says it is the go arm's status-strand work, that the caller has already brought reed up, and that only `start` calls it.

In `start.go`'s `RunE`, at the spot `c.ensureStatusStrand()` sits today (under the bootstrap lock, before the watchdog spawn and the driver spawn), write:
1. `c.reed.Up()` unconditionally; a failure releases the bootstrap lock and writes the error envelope, as today's `ensureStatusStrand` failure does.
2. Then branch on `mustUseLLMDriverArm(driver)`: the llm arm calls `removeStatusStrands(c.reed.Status, c.reed.RemoveStrand)`, which never fails `start`;
   the go arm calls `c.ensureStatusStrand()` with today's failure handling.

The llm driver strand is therefore added later, in `runDriverSpawnAndWait`, into a session that no longer holds the status strand.
The bootstrap lock's hold region is unchanged.

Reword the prose that says `start` always ensures the status strand:
- the file header comment;
- `mustUseLLMDriverArm`'s doc comment, which now also selects step 4's strand branch, not only step 5's spawn;
- the `RunE` comment naming the returned driver "step 5's branch condition" — it is now the branch condition for step 4's strand branch and step 5's spawn, still read only once;
- the step 4 comment ("ensure the reed substrate and its status strand");
- the watchdog-placement comment, whose second point justified the call's placement by `ensureStatusStrand` being shared with `step` — it now says the spawn sits after the strand branch, outside it, so both arms reach it, and `step` does not spawn the watchdog;
- the `Long` help text's step 2, which becomes: ensure the worktree's tmux session is up; on a go-driven run ensure its status strand exists, and on an llm-driven run remove any status strand the session still holds; then spawn the per-hub watchdog daemon.

In `start_watchdog_test.go`, correct the file doc comment: the watchdog call now sits after `c.reed.Up()` (and the strand branch), which is what keeps it out of an offline test's reach.
In `integration_driverbootstrap_test.go`, correct the file header's reason for driving `startLLMDriverArm` rather than the full RunE: `start` spawns the status strand's watcher pane via `os.Executable()` only on a go-driven run, so name that hazard as the go arm's and keep the rest of the rationale.
In `smoke_starttail_test.go`, correct the header comment the same way, including its clause that the attach tail leaves the status strand as "the only non-driver strand" — on the unseeded fixture, which `resolveSeedDriver` seeds `llm`, the attach tail adds neither a loom-operator strand nor a status strand — and change `TestSmokeStart_AttachTailAddsNoOperatorStrand`'s status-strand assertion: the fixture is unseeded, so `resolveSeedDriver` seeds it `llm`, and the attached start must leave 0 strands named `statusStrandDisplayName` (keep the `loom-operator` assertion); update the test's doc comment to match.
In `smoke_test.go`, `TestSmokeBootstrap_BringsUpSessionStrandAndDriver` and `TestSmokeBootstrap_SecondInvocationDoesNotSpawnASecondDriver` assert one status strand and a detached Go driver, which are go-driver properties: call `seedGoDriverRun` before the first `start` in each so they keep testing the go arm's keep/add behaviour.
In `smoke_drivingsurface_test.go`, add `TestSmokeStart_LLMRunRemovesEveryStatusStrand`: on an llm-seeded fixture, bring reed up through the probe engine and add two strands with `statusStrandAddSpec` over a long-lived shell command, run `lyx loom start --no-attach` (the providerless shuttle config makes the driver launch fail, which the test ignores: the removal runs before the driver spawn), and assert 0 strands carry `statusStrandDisplayName`.
Also add `TestSmokeStart_FailedReedUpRefusesOnEitherDriver`: a table over a go-seeded and an llm-seeded fixture, each with the invalid-`mouse` reed config Card 2's `TestSmokeStep_FailedReedUpRefusesWithBootstrapKind` uses (factor the fixture into one file-local helper shared by both tests);
`lyx loom start --no-attach` exits 1 with an `ok: false` envelope, and afterwards `findWatchdogPIDs` for the fixture hub and `findDriverPIDs` for the worktree both return none, pinning that a failed `Up` refuses before the watchdog and driver spawn on either arm.
This test needs no tmux; the removal test calls `tmuxBinaryPath(t)` first so a machine without tmux skips it.

In `smoke_driverstrand_test.go`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` is the one llm-seeded start whose driver strand really spawns, so after each of its three bootstraps also assert `statusStrandCount(t, eng, statusStrandDisplayName) == 0` beside the existing driver-strand count, and name that property in the test's doc comment and the file header.
This pins, in one session, that an llm-seeded start over a strand-free session adds no status strand while its driver strand still spawns.

In `CONSTRAINTS.md`'s Driver Choice Single-Site Invariant, change the opening sentence's "the branch on it selects a spawn and nothing else" to "the branch on it selects the run's driving surface — which driver spawns and whether the session carries the loom status strand — and nothing else."
The first sub-bullet and `driverFieldReadCarveOuts` stay unchanged: the read stays inside `internal/loomcli`.

In `docs/overview.md`, the `start` description's second step becomes: ensure the worktree's tmux session is up, keep or add the status strand on a go-driven run and remove any status strand on an llm-driven run, then spawn the per-hub watchdog daemon, best-effort.
Add one sentence that `step` brings the tmux session up and never adds or removes the status strand.
The overview describes `start` a second time, in its "the bootstrap" bullet ("adds the `lyx loom status` strand (replaced in its slot when the live one was launched from a different `lyx` build)"); rewrite that clause too: on a go-driven run it keeps or adds the status strand, replaced in its slot on a build change, and on an llm-driven run it removes any status strand the session still holds.

**ImpactSummary:** `lyx loom start`/`lyx start` on an llm-seeded or unseeded run stop adding the status strand and remove any left over; go-seeded runs keep today's behaviour; smoke expectations, one invariant sentence and the overview follow.

**Commit:** 3: loomcli: start chooses the status strand by the recorded driver
