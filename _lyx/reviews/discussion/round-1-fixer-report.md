# Fixer report — discussion, round 1

Every finding's premise was checked against the code and holds; all eight are fixed.
`lyx loom validate-discussion` passes over the edited pair.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Premise confirmed: `internal/shuttleengine/template.yaml` carries `claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles`, `claudeengine`'s `resolvePromptCacheTTL` resolves the TTL per role, and the writer runs under role `discussion`.
  D1 now tells the ping interval to the gate as a `GateConfig` duration whose zero turns pings off.
  A provider-neutral `shuttleengine.Engine` method answers a session's idle cache lifetime, implemented in `claudeengine` from the resolved TTL.
  The loom wiring sets the interval to that lifetime less one minute, and to zero when the lifetime reaches the wait bound, so a `1h` writer is never pinged.
  D1's Rationale weighs the `1h`-role alternative (about 0.75 of the context extra on every run against at most 0.3 for three pings on the runs that need them) and scopes its prices to the 5m TTL.
  Added a `PATTERN-shuttle-provider-seam` Constraint, an acceptance line for the 4-minute default and the unpinged `1h` writer, a plan-writer note, and two Rejected alternatives in the support log (the `1h` role, the hard-coded constant).
  The Auto-mode assumption now names the one-minute margin as the writer's pick.
- **F2 (MEDIUM), fixed as proposed.**
  Premise confirmed: `finalize` calls `evaluateGate(true)` once, and `SingleLLMProducer.mapOutcome` maps a failed `Result.Gate` to `Stuck`; Discussion-Write has no `on_stuck`.
  D2's Bound now states the reach: a mid-turn tree that fails the mechanical entry halts the run `Stuck` with the request expired, where a boundary would re-prompt.
  It records the decision to accept that, since the run deadline's finalize reaches the same end for a writer stuck mid-turn.
  The Open risks line points at the Bound, the acceptance line says `done` or `Stuck` by the mechanical entry, and the narrowing is a Rejected alternative.
- **F3 (MEDIUM), fixed as proposed (narrowing branch).**
  Premise confirmed: `verdict.json` holds kind, recorded-at, consumed and superseding only.
  D11 now claims only what the two files show: the writer's stamp and a verdict recorded after it.
  The landing description must name the gap, which is a new Open risk addressed to the parent reviewer; the acceptance line is narrowed to match.
  The transcript route is a Rejected alternative, since no lyx verb reads it.
- **F4 (LOW), fixed as proposed.**
  Premise confirmed: `closures.pending` always sets the `lyx loom review notify` way-forward.
  D1 now gives a ping its own failed-send way-forward (nothing needs doing; the next ping retries) that never names `notify`; an acceptance line pins it.
- **F5 (LOW), fixed as proposed.**
  Premise confirmed: `gate()` returns on `holding(r)` before any carry.
  D1's Bound now says no ping is carried during the reviewer-liveness hold, with the reason; the acceptance line lists the hold.
- **F6 (LOW), fixed as proposed.**
  Premise confirmed: `resolveReviewTarget` resolves a slug from any worktree.
  D5 now keys on whether a slug was given: with a slug the refusals name `lyx batten status <slug>` run from the prime, with none `lyx loom status`; title, Rationale and acceptance line follow.
- **F7 (LOW), fixed as proposed.**
  D10's in-flight sentence now names the run whose reject is recorded but whose rewrite has not passed the gate, and says a run already past Discussion-Write keeps its unreviewed rewrite.
- **F8 (NIT), fixed as proposed.**
  Checked: expiry is tested against opened-at before any carry, and pings run from delivered-at, so the last is at minute 56.
  D1's Rationale and both support-log mentions say 14.

## Disputed

None.

## Deferred

None.
