# Card 14 — adopt-spawns-check

**Edit:**
- `internal/adoptcli#suiteStart`
- `internal/adoptcli#suiteArgs`
- `internal/adoptcli#startSuite` -> `func startSuite(dir, outPath string) (int, error)`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#Command`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#TestStartSuiteStartsNothingUnderTest`
- `internal/adoptcli/doc.go`
- `docs/overview.md`

**Create:**
- `internal/adoptcli#checkAlive`
- `internal/adoptcli#ensureCheck`
- `internal/adoptcli#TestEnsureCheck`
- `internal/adoptcli#TestCheckAlive`

**Uses:**
- `internal/adoptcli#ReadCheck`
- `internal/adoptcli#CheckRecord`
- `internal/adoptcli#checkRecordName`
- `internal/adoptcli#checkOutName`
- `internal/adoptcli#suiteLogName`

**Intent:**
Decision 10's spawn: `lym adopt` starts the hidden check, detached in the prime, in place of the bare `lym gate test --gate finalize --wait`.

- `suiteArgs` becomes `adopt check`.
- `startSuite(dir, outPath)` starts `<lym> adopt check` detached in `dir` with its stdout and stderr on `check.out`, created for append, never `suite.log`, so the gate's output alone reaches `suite.log`, a losing check's envelope goes to `check.out` and a panic of the check is on disk; it no longer truncates `suite.log`, which the check does under its lock. The spawn is logged through `logger` (PATTERN-spawn-observability), and it still starts nothing under `go test`. It returns the started pid.
- `suiteStart` gains `Record string` with the tag `json:"record"`, the path of `check.json`, so the envelope's `suite` names the check's pid, the log and the record.
- `checkAlive` reports whether the record's pid is alive by `proc.IsAlive` and the live process's `proc.StartTime` equals the record's `StartTicks`; a record without ticks, or a platform where the start time is unreadable, reads alive on the pid alone.
- `ensureCheck` reads `check.json` in `dir`: when it reads `running` and `alive` passes, it spawns nothing and returns the running check's pid with the log and record paths; otherwise it calls `spawn` and returns the new pid with the same paths.
- `hubDepsAs` wires `StartSuite` to `ensureCheck` over `fabricengine.AdoptScratchDir`, `checkAlive` and `startSuite` in the prime's anchor; `adoptDeps`' doc names the check instead of the finalize gate.
- `adopt`'s warning for a check that does not start becomes `the adopt check was not started (<error>); way forward: run lym fabric reconcile in the prime, then lym gate test --gate adopt there`, since no reconcile of the hub's config ran and `lym gate test` runs none, while `lym fabric reconcile` runs card 11's hub-base seed of `adopt` first.
- `Command`'s Long text, step 8, says the adopt check starts detached in the prime, reports each tier to the orch and shows in `lym beam status`.
- `doc.go`'s step 6 is rewritten for the check: the spawn, `check.out`, the alive skip, the warning and the bound that between a spawn and the child's lock `lym beam status` can show the previous record and a second `lym adopt` can spawn a check that loses the lock and exits, either envelope's pid possibly naming the check that lost, both naming the same log and record. `docs/overview.md`'s adopt entry names the `adopt` gate and the check in place of the `finalize` gate.

Tests:
- `TestEnsureCheck` (tier0, temp directory, a counting `spawn` fake): no record spawns once and names the new pid, `suite.log` and `check.json`; a running record that reads alive spawns none and names its pid; a running record that the `alive` seam reads dead spawns once; a green record spawns once; a failed spawn returns its error.
- `TestCheckAlive` (tier0, over this test process): a record with `os.Getpid()` and its own `proc.StartTime` reads alive; the same pid with other ticks reads dead, the reboot-reused-pid case; the same pid with no ticks reads alive on the pid alone; a pid no process holds reads dead.
- `TestAdopt`'s `FailedSuiteStartIsTheWarningNamingTheGate` row expects the new warning text, naming `lym fabric reconcile` before `lym gate test --gate adopt`.
- `TestStartSuiteStartsNothingUnderTest` passes a `check.out` path and asserts no directory is left behind, as now.

**ImpactSummary:** The adopt's last step and its warning text change for one seam, `StartSuite`, wired in `hubDepsAs`; the envelope's `suite` gains `record`.

**Commit:** 14: Start the adopt check from lym adopt and skip a live one
