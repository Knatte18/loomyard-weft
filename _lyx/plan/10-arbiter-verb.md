# Card 10 — arbiter-verb

**Create:**
- `internal/loomcli/arbiter.go`
- `internal/loomcli/arbiter_test.go`
- `plan:internal/loomcli#arbiterRun` -> `type arbiterRun interface`
- `plan:internal/loomcli#arbiterShuttle` -> `type arbiterShuttle interface`
- `plan:internal/loomcli#arbiterDeps` -> `type arbiterDeps struct`
- `plan:internal/loomcli#arbiterInput` -> `type arbiterInput struct`
- `plan:internal/loomcli#arbiterVerb` -> `func arbiterVerb(out io.Writer, deps arbiterDeps, in arbiterInput) int`

**Edit:**
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomcli#arbiterEligibility`
- `plan:internal/loomcli#arbiterPrompt`
- `plan:internal/loomcli#arbiterSpec`
- `plan:internal/loomengine#ListArbiterAttempts`
- `plan:internal/loomengine#ReadArbiterVerdict`
- `plan:internal/loomengine#WriteArbiterMarker`
- `plan:internal/shedengine#CheckGotoTarget`

**Intent:**
Implement the body of `lyx loom arbiter` (D1, D5–D9) over injected seams, so every branch is a Go-tier test with a fake shuttle and clock (`PATTERN-test-speed`).

`arbiterRun` is the handle of a started arbiter: `Wait() (shuttleengine.Result, error)`, `Stop() error` and `StrandGUID() string`.
`arbiterShuttle` is `Start(spec shuttleengine.Spec) (arbiterRun, error)`, `AttachIfLive(spec shuttleengine.Spec) (shuttleengine.Result, bool, error)` and `StopStrand(guid string) error`; card 11 adapts `*shuttleengine.Runner` to it.
`arbiterInput` is the run id, the stop envelope path and the optional report path.
`arbiterDeps` carries: the status read; the settings (`loomengine.ArbiterSettings`); the arbiter directory, the attempt-directory constructor and the marker path; the producers for `CheckGotoTarget`; closures for the Bouncer escalation (`shedadapters.PendingEscalation` at a Bouncer row, false elsewhere), the pending circling decision, the Publish verify-failure target and the interrupt policy of a row; the prompt inputs that do not vary per attempt (stencils dir, slug, worktree, decision record and support log paths); the shuttle; `now`; and the relaunch line's run id.

`arbiterVerb`, in order:

1. Reads the status file and the envelope at the given path (`kind`, `transient`); a missing status, an unreadable envelope or a status read error is refused.
   The stop key is `loomengine.StopKeyOf(status)`.
2. Lists the attempts; the newest attempt whose stop key equals the current one decides the call, so a re-run spawns nothing (D1):
   - applied (`loomengine.ArbiterApplied`) with a `resume` verdict: refused, "an applied resume left the stop key unchanged", never claiming recurrence (D7);
   - a verdict on disk: that verdict, validated as in step 5;
   - no verdict and the marker names this attempt: wait as in step 4 through `AttachIfLive` with `Timeout` the time left to the marker's deadline; not found reads as `hand-on`, cause `died`;
   - otherwise `hand-on`, cause `missing`.
3. With no attempt for this stop: refuses an ineligible stop through `arbiterEligibility`, then refuses `MaxSpawns` 0 ("the arbiter is off") or attempts already at `MaxSpawns` ("the spawn budget is spent"), spawning nothing.
   Otherwise it writes the new attempt's `attempt.json` first (number one past the highest, stop key, the status's `pause_before` and `pause_after`, spawn time, deadline as spawn time plus the timeout, envelope and report paths), so the slot counts even if the spawn fails; renders the prompt with the verdict path and the escalation brief at a Bouncer stop, saves it as `prompt.md`; logs the spawn through `internal/logger` (`PATTERN-spawn-observability`); starts the run; records the strand guid in `attempt.json` and writes the marker (attempt, guid, deadline).
4. Waits on the run's `Wait`, whose wakes shuttle bounds (`PATTERN-daemon-wakeups`).
   An `OutcomeTimeout` or any non-done outcome without a verdict calls the run's `Stop`, or `StopStrand` with the marker's guid on the re-run path, shuttle's stop verb (`PATTERN-shuttle-stop`), logs it, and reads as `hand-on` with cause `timeout` or `died`.
5. Validates a verdict: unparseable reads `hand-on` with cause `unparseable`; a `resume` reads `hand-on` with cause `precondition` when, at an `awaiting` stop, no circling decision is pending, at a `blocked` stop it names no goto target, or any goto target it names fails `CheckGotoTarget` against the current status.
6. Removes the marker, writes `envelope.json` in the attempt directory and prints the same envelope: `verdict`, `reason`, `question`, `goto`, `verbs`, `cause` (empty for an agent's own verdict), `attempt`, `attempt_dir`, `verdict_path`, and for a `resume` only, `relaunch`: `lyx shed step <run-id> --until-stop --arbiter-attempt <N>`.

Each refusal is an error envelope whose way forward is the hand-back to the run's parent, with `lyx loom status <run-id>` to see the stop.
Bound: the verb spawns at most `arbiter_max_spawns` agents per run, counted from durable records so no restart resets it, and moves no run itself.

`arbiter_test.go` covers the verb through fakes: one table-driven test for the spawn path (each eligible stop spawns exactly one run whose spec, attempt record, prompt and marker carry the expected values; an ineligible stop, a budget of 0 and a spent budget spawn nothing), one for the re-run path (a live attempt attaches without starting, a finished unapplied verdict returns without starting, an applied `resume` refuses, an attempt for another stop key does not match), and one for verdict outcomes (valid `resume` with its `relaunch` line, `hand-on`, missing, unparseable, each missing precondition, and a timeout calling `Stop` and reading `hand-on`).
A budget counted from attempt records a fresh `arbiterDeps` reads again shows the count surviving a restart.
Raise `internal/loomcli`'s row in the test budget by the tests added.

Add loom-section rows to `contracts/specs/refusal-spec.md` for `arbiter: status or envelope unreadable` (transient), `arbiter: off`, `arbiter: spawn budget spent` and `arbiter: an applied resume left the stop key unchanged`, each quoting its way forward.

**ImpactSummary:** a new unexported verb body with no caller until card 11; refusal-spec and the test budget gain rows.
