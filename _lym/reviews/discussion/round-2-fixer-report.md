# Fixer report — discussion round 2

Every finding's premise was checked against the code (`internal/configengine/config.go:274`, `internal/dobbyengine/doc.go`, `internal/dobbyengine/config.go:64`, `internal/gatecli/testmap.go:471`, `internal/beamcli/dobbydeps.go:599-614`, `internal/fabriccli/anchordeps.go:459`) and holds.

## Changes

- F1 (fixed as proposed, first option): D8 now says `configengine.Spec` gains a per-layer migrate hook that `configengine.Load` applies in memory before the merge and `DropRetired`.
  `dobbyengine` owns the rewrite, which its `ConfigSpec` and beam's `Migrate` both call; the in-memory pass logs nothing.
  The Rationale says the off switch holds before the walk as after it.
  Added a D8 criterion: a prime dobby re-exec'd over `test_map_every: 0` with no walk run starts no pass.
- F2 (fixed as proposed): D1's Miss bullet says a whole rebuild's first written round starts from an empty map holding that round's results and a fresh spawn scan's verdicts, so the map at `H` holds only entries measured or scanned at `H`.
  The D1 Miss criterion adds that the map at HEAD after the first round holds no entry measured before HEAD.
- F3 (fixed as proposed, extended by the self-check): D8's second bullet says a `--upkeep` call whose registration fails reports it and exits before it plans, that the dobby logs that exit once per episode, and that the dobby starts no pass while a pass it started is its live child, registered or not.
  The Bound now names only an unregistered call naming no package and lacking `--upkeep`.
  Two D8 criteria cover the failed registration and the not-yet-registered child.
  The PATTERN-child-processes Constraint now names a `--upkeep` pass as the registered one.
- F4 (fixed differently): D9's retake tries the build lock without waiting, as proposed.
  When the lock is held, the pass skips the package as D1 skips a map whose lock is held, and goes on to its next job.
  The proposal had it release the slot and wait again under D9's rule.
  The self-check's fix-proposal lens showed that this stalls every other queued package behind one hand call's whole rebuild.
  It also takes and releases a slot each second.
  D1's existing skip for a held lock avoids both.
- F5 (fixed as proposed): dropped the first Open risks bullet.
  In support-log.md, Q15 is resolved, citing the Added after Discussion entry of 2026-11-11.
  Both Turn 5 lines that kept the question under Open risks now say it was settled.
- F6 (fixed as proposed): D1's Current and Affected bullets except the outcome entries D4 drops, which are dropped and queued.
- F7 (fixed as proposed): D8 classifies `test_map_upkeep` as `configengine.CategoryNone`, as `test_map_every` is today.
- F8 (fixed as proposed): the first D8 criterion is qualified with "on a hub with two or more slots".
- F9 (fixed as proposed): D1's marker paragraph says every `Commit`-moving write also sets the marker's environment stamp (D6) to the current one.

## Self-check

- fix-claims, MEDIUM: beam's `LoadConfig` cannot reach the layers before `DropRetired`. Fixed as proposed: D8 names a per-layer migrate hook on `configengine.Spec`, applied inside `Load`.
- fix-way-forward, MEDIUM: the same finding, with the import-cycle point. Merged with the above; the rewrite lives in `dobbyengine`.
- fix-proposal, MEDIUM: the same finding, plus repeated Warn logging on every load. Merged with the above; the in-memory pass logs nothing.
- fix-consistency, MEDIUM: the D8 Bound contradicted the Constraint "a pass stays registered". Fixed as proposed: the Constraint names a `--upkeep` pass.
- fix-consistency, LOW: the D8 criterion tied the live-child rule to a child that had already exited. Fixed as proposed: split into two criteria.
- fix-consistency, LOW: support-log Turn 5 still held the parent question under Open risks. Fixed as proposed.
- fix-proposal, LOW, and fix-way-forward, LOW (merged): a persistent registration failure restarts the pass every tick, silently. Fixed as proposed: the call reports the failure, and the dobby logs it once per episode and retries on a later tick.
- fix-proposal, LOW: F4's wait stalls the pass behind one hand call. Fixed as proposed: the pass skips the package per D1 (see F4 above).

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
