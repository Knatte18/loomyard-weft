# Driver arbiter: a cold agent settles stops the driver cannot

A run's driver that reaches a stop it cannot settle now spawns a cold arbiter agent on a stronger model.
The arbiter decides the stop from the task's own records, so the driver's parent (today the hub orch) is woken only for a call that needs an operator decision or that the arbiter is unsure of.
Fixed-rule escalations, such as a review escalation that is accepted unless a finding makes the design wrong, no longer cost the orch a full-context wake.
Loom and darn runs both get it, since they share the shed driver stencil.

## What changes

- New verb `lyx loom arbiter`, run by the driver as a background job (hidden from the orch's command index).
  It checks eligibility, spawns the arbiter as a shuttle run in the pair's reed session, waits for the verdict file and prints one envelope with the verdict.
  The verdict is `resume` or `hand-on`, with a reason.
- Eligible stops, a closed list: an `awaiting` Bouncer escalation with no circling decision recorded; a `blocked` stop; a `producer`, `bootstrap` or reinvoke-policy `interrupted` error after the repair fork escalated, and a transient error that failed twice.
  Everything else is refused with a way forward naming today's hand-back.
- The arbiter may only run `lyx loom circling accept|continue` and `lyx loom decision add --by arbiter`.
  It never runs `goto`, resume, approve, reject, review or any mutating git command, and never messages a session.
- On `resume`, the driver re-launches the loop naming the attempt (`--arbiter-attempt`).
  The re-launch takes the run lock, checks that the stop key and the pause fields are what the verb recorded at spawn, writes an applied mark, performs the verdict's goto if it names one, then steps the run.
  On `hand-on`, a missing or unparseable verdict, an unmet `resume` precondition, a timeout or a spent budget, the driver parks and notifies its parent as today, with the arbiter's reason and verdict path in the stop report.
- New `loom.yaml` keys: `arbiter` (default `opus[high]`), `arbiter_timeout_min` (default 30) and `arbiter_max_spawns` (default 3 in the template).
  `arbiter_max_spawns: 0` or an absent key turns the arbiter off, so every eligible stop escalates as before; a negative value is refused naming the key.
- `lyx loom decision add --by arbiter` is admitted from the run's own arbiter strand only.
- Batten holds its state-changed notice while a live arbiter marker is within its deadline, and its strand revive never revives an arbiter strand.
- The orch stencil tells the orch that an escalation from a run whose arbiter handed on carries the arbiter's verdict path.
- Touched modules: `internal/loomcli`, `internal/loomengine`, `internal/shedengine` (`Relaunch`, `CheckGotoTarget`), `internal/shedverbs`, `internal/shedrun`, `internal/shedadapters`, `internal/battenshed`, `internal/battencli`, the arbiter and driver stencils, `refusal-spec.md` and the package docs.

## Design decisions to review

- The spawn budget counts attempt records under the run's durable directory, so a driver or batten restart never resets it.
- A stop that reaches the verb with the stop key (`current_producer`, history length, state) of an applied `resume` attempt is handed on, so one recurring stop earns one re-launch.
  The refusal says the applied resume left the stop key unchanged; it does not claim recurrence, because a driver that died after the mark looks the same.
- The goto lives in the re-launch, not the arbiter, because `lyx loom goto` clears the pause fields and would erase a human `pause` before the state check could see it.
  An orch `goto` or `pause` during a live arbiter therefore costs the arbiter's settlement and goes to the parent; a plain `pause` landing between the verdict and the step survives the re-launch.
- A `blocked` Publish halt whose verify-failure record names a target is refused in every run, watched or not, because the verb never reads the watched marker and must not race batten's own Publish retry.
- Whether a repair fork escalated is not in the status file, so that half of eligibility is a driver-stencil bound: the stencil runs the verb at an error stop only after a fork's escalation.
  A driver that skips the fork gets at most one re-launch per budget slot.
- The `resume` re-launch reverses, for this one case, the driver stencil's rule that a `blocked` stop is never repaired, and it is allowed over a spent repair cap.

## Check by hand

- Not run: a live arbiter end to end (real shuttle spawn on `opus[high]`, verdict, re-launch); the tests use fake shuttle and clock seams.
- `arbiter-verb-unreadable-input-no-test`: `internal/loomcli/arbiter.go` has no test for the verb's handling of an unreadable input file (status, envelope or escalation); confirm it hands on with a way forward.
- Confirm the `internal/loomcli/arbitereligibility.go` reading of a non-empty transient class, which makes any error kind eligible, is the intended one.
- `internal/testkit/envkit`: `FullEnv` now stubs `InnerRun.ArbiterInFlight` to report no arbiter; check the stub is a fair stand-in for the integration tier.
- A driver that dies mid-arbiter and is never revived leaves the verdict unapplied until batten's hold expires and the parent is notified.
- A wrong `circling accept` lets an unconverged segment pass; the verdict and the decision entries are committed, so the decider is visible.
- `contracts/stencils/orch/orch-template-role.md` is also edited by the `parent-review-wait` and `orch-split` tasks; expect a merge there.
- Test budget rows in `cmd/lyx/testdata/test-budget.yaml` rose for `internal/loomengine`, `internal/loomcli` and the two batten packages.
