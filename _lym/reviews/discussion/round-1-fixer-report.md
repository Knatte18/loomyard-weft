# Round 1 fixer-report — discussion decision record

Every change is in `_lym/discussion/decision-record.md`; the support log is unchanged.
F10 is `decision`-class and went to the parent, so it is left out here.

## Changes

- F1 (fixed as proposed): D8's last bullet now names beam's `configreg.Module.Migrate` in place of a `configreg.Reinterpretation`.
  It cites the walk order: `ReconcileHubWideAt`'s `pruneSparse` runs before `applyReinterpretations`, and `Migrate` runs before the prune.
  D8's Rationale and the PATTERN-config-layers Constraint now say "migration".
  The D8 criterion names the reconcile walk's migration.
- F2 (fixed as proposed): D1 now says that under every verdict but Miss the plan queues each declared mapped test with no entry,
  and that spawn-marked tests are excepted because they have an entry.
  A Current map with such a gap is re-stamped and has those tests queued.
  D4's outcome-entry bullet now says that dropping an outcome entry queues that test.
  The D4 yield criterion adds that the next pass after one that ended still queues the test.
- F3 (fixed as proposed, first option): D8 says the same migration runs on the prime's `_lym/config/beam.yaml`
  and on beam's local override file when `configsync.ReconcileAll` reconciles those layers.
  `reconcileSparseFile` runs `m.Migrate` for both layers (`configsync.go:532`, `:674`).
  The D8 criterion covers those files.
- F4 (fixed as proposed, second option): D8's start bullet says the dobby starts no pass on queued work alone
  while the hub's slot count is below two.
  A D8 criterion says that on a one-slot hub a pass with queued work starts no second pass at the same HEAD.
- F5 (fixed as proposed): D9's yield bullet releases the interrupted package's build lock and takes it again with the slot.
  D9's Bound says a hand call's wait for a build lock ends within one pass job,
  and a D9 criterion covers a hand call that names a package whose job was yielded.
- F6 (fixed as proposed): D10's bullet and criterion select tests that call `t.Parallel()` as a top-level statement
  before any other statement but `tiergate.Require`.
  The bullet cites PATTERN-test-speed.
- F7 (fixed as proposed): D11 is rewritten.
  `fabriccli`'s `dobbyPassInFlight` no longer counts a registered `runproc.RoleTestMap` process,
  and the anchor verb's `EndDobbies`, through `runproc.EndSparing`, ends the pass.
  The claim that the dobby ends it is dropped.
- F8 (fixed as proposed): D8 names the hidden `--upkeep` flag that the dobby passes and that makes the call a pass,
  and states what a hand caller can do with it and what bounds that.
  Only a call carrying `--upkeep` reads or writes the exit record.
  D9 says a call that names no package and lacks the flag runs as a pass but leaves the exit record as it was.
  The D8 criterion now reads "a hand call without `--upkeep`".
- F9 (fixed as proposed, first option): D2's non-Go bullet and D3 define a package's own non-Go files
  as every tracked file under its directory tree down to the next package directory, `testdata` included.
- F11 (fixed as proposed): D6's stamp adds a digest of the hub `env.yaml` variables the call passes its children,
  and the D6 criterion covers that case.
- F12 (fixed as proposed): D1 and D8's start bullet now count a map skipped for a held lock as queued work.
- F13 (fixed as proposed): D7's self-check checks own-package block keys against the fresh build's block table
  and linked functions against the fresh build's function set for each linked package.
- F14 (fixed as proposed): D8's carve-out defines a recorded failure as D5's failure field
  or a failure-only map's `BuildFailure` with its `Commit`.
  D12's queued package count is the queued work left when the pass ended.
- F15 (fixed as proposed): D13 adds that the parent carries the change for each of the two notes into the note's body.

## Self-check

- fix-way-forward, MEDIUM: a bare call on a hub with fewer than two slots builds nothing, but D9's Bound said hand calls build maps.
  Fixed: the Bound now reads "only hand calls naming packages build maps".
- fix-way-forward and fix-proposal, MEDIUM (merged): a hand `--upkeep` call is unregistered, so the dobby could start a second pass beside it.
  Fixed: under `--upkeep` the verb registers its own process as `runproc.RoleTestMap`,
  in place of the dobby's registration of its child,
  and refuses to start while another such process lives.
- fix-consistency, LOW: nothing said what a call does that names packages and carries `--upkeep`.
  Fixed: `--upkeep` refuses package arguments.
- fix-consistency, LOW: D1 stated twice that the plan holds no gate slot.
  Fixed: the new sentence reads "The plan itself builds nothing and runs no test."
- fix-consistency, LOW: D5's no-requeue rule did not reach the failure-only map that D8 counts.
  Fixed: D5 now defines a recorded failure as the field or a failure-only map's `BuildFailure` with its `Commit`, and applies the rule to both.
  D8's carve-out refers to D5's definition.
- fix-proposal, MEDIUM: F12 together with D8 restarted a pass on every tick while a hand call held a lock.
  Fixed: a pass now plans a lock-skipped map again once the lock frees, under D9's wait rule.
  Only a map that is still skipped when the pass ends counts as queued work.
- fix-proposal, MEDIUM: a yielded pass that retook the lock could overwrite a map a hand call had written.
  Fixed: after it takes the lock again, the pass reads the map again and plans it afresh at `H` before it runs another test.
- fix-proposal, MEDIUM: F9's rule missed Go files under `testdata`.
  Fixed: D2 and D3 exclude only the package's own Go files and include Go files under `testdata`.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
