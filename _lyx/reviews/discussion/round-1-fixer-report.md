# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING): premise confirmed: `verifytree.Verify` acquires a slot and passes `lease.Env(env)` to the verify command, and `internal/gatecli/run.go` takes the inherited branch on `pool.Inherited`.
  D7 now keys the exemption on a variable `lyx gate test` sets only in its own go test child's environment, states that an inherited slot does not turn the watchdog off and that the slot holder is not always an outer `lyx gate test`, and restates the Bound to the outermost `lyx gate test`, acquired or inherited slot.
  The watchdog acceptance criterion now says "under an enclosing `lyx gate test`", and a new criterion pins inherited-slot-without-enclosing runs the watchdog while nested does not.
- F2 (MEDIUM): D7's threshold is now at least 90 percent of one core over every sample period, with the reason; a sample under that threshold restarts the window.
  The Open risk on livelocks and the Auto-mode assumption use the new threshold; the integration criterion spins one goroutine.
- F3 (MEDIUM): D6 names the window and `busy_quit_sec` raised through `lyx config gate` from the prime when the watchdog line matched, in the report, findings and stuck reason; the refusal-spec row change carries that way forward; the stuck-reason criterion covers a watchdog quit.
- F4 (MEDIUM): D5's Bound says any exit code of 128 or above reads as a hang whether or not a signal caused it, and skips only the rerun.
  D6 names the matched marker or exit code and claims stacks in the log only when a timeout or SIGQUIT marker matched; a bare high exit code is reported as killed or exited with that code.
  The stuck-reason criterion covers that case.
- F5 (LOW): D3's Bound states that a map value replaced under the same map key keeps the old value's unknown keys; the Open risk on fresh records carrying old keys names that case.
- F6 (LOW): D9 says the scan covers only `encoding/json`'s `DisallowUnknownFields`, that `yaml.v3`'s `KnownFields` used by the strict contract decoders is outside it, and that a future strict agent-written JSON contract validates its keys some other way.
- F7 (LOW): D7's Bound says a `-bench` or `-fuzz` run is bounded only by go test's own `-timeout`, as before.
- F8 (LOW): the `proc-child-lifetime` sentence is deleted from D7's Rationale and recorded as a board-fit bullet in `support-log.md`'s Interview entry; the Decision's placement stays.
- F9 (NIT): blank lines inserted before `### D8` and `## Open risks`.

## Disputed

None.

## Deferred

None.
