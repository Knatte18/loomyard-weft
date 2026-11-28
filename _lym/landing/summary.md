# Keep a beam member's mail and calls with its beam's orch

A beam's orch owns its members' runs, but was often treated as not their parent.
A member's asks and notices went to the prime's orch,
the beam orch's answers and decisions were refused,
its tackler prompt said no parent was recorded,
and it could not lift a tender pause on its own member.

## What changes

**Notice routing.**
The orch a walk up the beam chain starts at now takes the mail while reed still holds its strand,
even when the liveness probe reads it as down; creel delivers it once the orch is live.
An unstarted or stopped orch still passes it to the nearest live orch above, else the prime's.
The rule is `orchroute.Destination`, and notices, asks, `creel post --parent` and dobby's lines follow it.

**Caller checks.**
`loom decision add`, `replace` and `store`, `loom parent-review answer`, `drift amend` and `drift defer`
admit orchs on the run's beam chain without asking whether they are live.
For a run's record these are the orchs above the pair.
For a beam's project record they are the beam's own orch and those above it.
An orch higher up the chain, the prime's included, is therefore admitted even while a nearer one is live.
When the chain cannot be read, the orch that receives the run's notices is still admitted.

**Prompts.**
The shed tackler and arbiter prompts name the run's parent orch, in loom and batten runs alike.
They use a new `parent-directive-handback` stencil,
which keeps the rule that a blocked role writes its report and stops.
A batten run whose parent name cannot be read logs a warning and renders the no-parent text.
The driver prompt is unchanged, since it already named the pair's recorded parent.

**Tender.**
A beam's orch may now `tender pause`, `resume` and `revive` its beam and every pair under it.
It may not act on the whole hub or another pair, and it may not `tender reap`.

Closes #232
Closes #231
Closes #236
Closes #252
Closes #251

## Design notes

- Routing turns on whether reed holds the strand, not on the orch state's record of it:
  stopping an orch removes the strand but keeps the record,
  so a record-based rule would park mail on a stopped or moved beam's orch.
- The liveness probe is unchanged, and why a running beam orch read as down in #232 is not known.
  If its strand was also missing from reed's state at the time, this change does not cover that case.
- The hand-back directive names the parent without promising that the stop reaches it,
  since a stopped orch's notices go further up.

## To check by hand

- The reported scenarios have not been replayed in a hub with a running beam:
  a member's PR-Review ask reaching the beam's orch and its answer being accepted,
  and that orch running `loom decision add` on the beam and `tender resume` on a member.
- `decisionOrchChain` in `internal/loomcli/decision.go`:
  check that its test covers both the project-record chain and the run-record chain.
- `checkParentCaller` in `internal/driftcli/amend.go`:
  check that `drift amend` admits a beam orch on the chain even when the notice destination is another orch.
- `contracts/stencils/parent/parent-directive-handback.md`:
  check that it names no orch as one the stop is sure to reach.
- `internal/hubgeom/doc.go`: check that it describes neither the old decision guard nor a consumer of `OrchChain`.
- No test asserts that `goDriverDepsFor` passes `c.tacklerPrompt()` into the tackler's dependencies;
  only what `tacklerPrompt()` returns is asserted.
- No repository-wide test run was made.

## Merge danger

**Door:** two-way
**Blast radius:** orch
