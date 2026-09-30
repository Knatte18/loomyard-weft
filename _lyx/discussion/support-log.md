# Support log — loom-start-refuses-mid-merge

## Interview

The session ran autonomously; every turn below is a self-posed question answered from the codebase or by auto-pick.

1. **Why did the conflicted tree get past `start`?**
   `start` runs origin resolution, seed, seed commit, substrate setup, then spawn — no merge or cleanliness check.
   The Preflight row (`preflightshed` over `preflight.Check`) would flag a `UU` path as `worktree-clean`, but it runs inside the driver at recipe head only, so a resumed run never meets it.
   The seed commit (`CommitAnchoredPaths`) is weft-only and does not consult the fabric merge record, so it did not stop the incident either.

2. **Where should the gate live?**
   Picked `lyx loom start`.
   `internal/preflight` would also reach the Preflight row, and `step` cannot be guarded because Landing's `mergeresolve` runs mid-merge inside a step.

3. **Should `start` refuse unconditionally when mid-merge?**
   No.
   Landing's conflict session keeps the pair mid-merge with a live driver, and an operator re-attaching with `start` must not be refused.
   Picked: refuse only when this invocation would spawn a driver or resume a parked one.

4. **Which merge states count?**
   Both a fabric-parked merge (record present) and foreign git-level state on either side (`MERGE_HEAD` or unmerged entries).
   The resolved-but-unconcluded shape (`MERGE_HEAD` only, or a record with every path staged) is the most dangerous because the tree looks clean, so it counts too.

5. **Which probe?**
   `MergeStateActive` is weft-only; `Fabric.MergeInProgress` is record-only and needs an open `*Fabric`; `foreignMergeStatePresent` is unexported and excludes the record.
   None covers the incident alone, so a new exported `l`-in probe combines the record check and the four git-level probes, returning a vocabulary-neutral classification plus conflicted paths.

6. **What does the refusal look like?**
   `output.ErrFields` with a new kind `merge_in_progress` beside `StartNotParkedKind`, and a `conflicts` field.
   Message by case: fabric remedy for a parked fabric merge, git-level remedy for foreign state, since the fabric verbs refuse foreign state.

7. **Ordering against the seed commit?**
   The check needs the strand/lock probe's decision, which runs after seeding under the bootstrap lock.
   The seed commit is weft-only and idempotent, so leaving it first is harmless.

8. **What about batten and ly-drive?**
   Batten classifies a child `start` refusal by kind (`childNotParked`); a new kind falls into its non-retryable path, assumed and flagged for the plan to verify.
   ly-drive is spawned by `start` and does not call it, so it is unaffected.

## Rejected alternatives

- **New `CheckID` in `internal/preflight.CheckResolved`.**
  It would reach the Preflight row and any future caller of `preflight.Check`, and `start` does not call `preflight` today, so it adds a second place to reason about without guarding the resume path by itself.
- **Guarding `lyx loom step` as well.**
  Landing's `mergeresolve` holds the pair mid-merge inside a step; a step-level refusal breaks Landing's own conflict flow.
- **Refusing unconditionally in `start`, even when only attaching.**
  Blocks an operator re-attaching to a live Landing conflict session.
- **Running the probe first, before seeding.**
  The refusal still has to wait for the spawn/resume decision, so an early probe only adds a second site; the seed commit it would pre-empt is harmless.
- **Reusing `MergeStateActive` or `Fabric.MergeInProgress`.**
  The first sees only the weft side, the second only the fabric record; the incident's shape (warp-side conflict under a fabric record) and the foreign shapes each slip past one of them.
- **Two kinds, one per case (fabric vs foreign).**
  Both are the same non-retryable class for a caller; the message carries the case difference.
- **Auto-aborting the merge from `start`.**
  Discards the operator's partial resolution; `start` refuses and the operator decides.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Gate location: `start`, `preflight`, or the Preflight row? — Resolved, auto-pick: `start`. Rationale: the only verb that puts a driver to work; `step` must stay unguarded for Landing.
- Q2 Refuse unconditionally or only on spawn/resume? — Resolved, auto-pick: only on spawn/resume. Rationale: live Landing conflict sessions.
- Q3 Which merge states count? — Resolved, auto-pick: fabric record plus foreign state on either side. Rationale: every shape leaves the driver on a half-merged tree.
- Q4 New probe or reuse? — Resolved, auto-pick: new exported `l`-in probe in `fabricengine`, reusing its internals. Rationale: no existing probe covers every shape, and `loomcli` cannot name the sides.
- Q5 Kind vocabulary? — Resolved, auto-pick: one kind `merge_in_progress` beside `StartNotParkedKind`. Rationale: callers branch on kind; retryability is the distinction that matters.
- Q6 Probe error handling? — Resolved, auto-pick: refuse on the envelope. Rationale: undetermined is not clean.
- Q7 Does batten retry unknown refusal kinds? — Open, for the plan: assumed non-retryable; verify in `internal/battencli/wire.go`.
- Q8 Crash-left Landing record blocking `start`? — Resolved, auto-pick: accept; the message names `--abort`. Recorded as an open risk.
