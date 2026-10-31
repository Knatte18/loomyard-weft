# Round 1 fixer-report

Every finding was checked against the code and every premise held.
All sixteen are fixed; none disputed, none deferred.
Files changed: `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.
`lym loom validate-discussion` passes on the result.

The parent's `Added after Discussion` entry, which arrived during the round ("The contract names a build segment, not a writer"), is left byte for byte.
Acting on it as the parent's instruction, the Goal, D1's contract clause, D1's acceptance row and D5's gate sentence now say build segment or Darn-Write row in place of writer row.

## Changes

- F1 (BLOCKING), fixed as proposed, with the fence form, and extended after the self-check: D4 sets the board body in one fenced `markdown` block whose backtick fence is one longer than the body's longest backtick run.
  It also states that `fencedLines` in `internal/discussionparser/append.go` takes CommonMark's fence-closing rule, since today it flips on any fence line whatever its length.
  The Bound line names the fence.
  The acceptance row adds three bodies: one with `## Decisions` and `## Open risks` headings, one with its own code block holding a `## Decisions` line, and one with an unclosed fence.
  Scope In's discussion-spec bullet names the fence rule.
- F2, fixed as proposed: D10 names the dropped half-finished checks beside the stat race guard (`internal/loomengine/coherence.go`).
  Its new Bound line says what they let through and how that is bounded, citing `waivesHalfFinished` and `WriteSeed`'s refusal of a disagreeing seed (`internal/loomcli/sharedbootstrap.go`).
- F3, fixed as proposed, using `RetiredKeys` with a refusing category and corrected after the self-check: D13 retires the keys under a category other than `CategoryNone`, so loads still drop a hub base value until reconcile.
  A check of their own refuses them with the way forward, run before `ValidateLocal`, in `PrepareRunSettings` and in loom's and darn's local-layer loads.
  D13 names the real refusal points: board `run_settings` writes, `profiles.yaml` writes and Seed-Child.
  It also states that an already-seeded pair whose local file holds a key is refused at load, naming the local `shed.yaml`.
  The acceptance row and the Open risks line on profiles match.
- F4, fixed with a pre-session check in Darn-Write, extended after the self-check: D4's in-flight paragraph now gives the handling for each parked row.
  - Preflight: Record-Write runs next.
  - Drift-Baseline or Drift-Await: the orch runs `goto --to Record-Write`; HEAD is still the branch point by `CheckMainCapture`.
  - Darn-Write: it refuses a missing record before its session. With no commit past the branch point the way forward is `goto --to Record-Write` then `resume`. With commits, it is `goto --to Record-Write`, `lym loom step`, `goto --to Darn-Write`, `resume`, which never passes Drift-Baseline.

  The acceptance row and the Open risks line match.
- F5, fixed as proposed with a stop marker: the writer writes a stop marker in the run's directory.
  Darn-Write clears the marker before each spawn and reads it once the session returns, after a Done handoff or a gate-failed Stuck alike, so a stop report that a gate refuses still halts on the description.
  The marker belongs to one spawn, so a respawn with earlier commits still halts.
  The Bound line and the acceptance row match.
- F6, fixed as proposed: D6's halt names `lym loom decision add`, run by the orch or the operator, then `lym loom resume`, so the answer reaches the writer and the perch as an `Added after Discussion` entry.
  The acceptance row adds this.
- F7, fixed as proposed: Record-Write writes whichever file is missing and never overwrites one that exists, then commits an uncommitted pair.
  The Bound line and a crash-between-writes acceptance row are added.
- F8, fixed as proposed: D12 names the `tapestry` module's config load as the place the plan perch is checked.
  It loads `loom.yaml`'s run-wide budget keys and runs `reviewbudget.CheckResolved` over the plan perch.
  The acceptance row adds a refused `tapestry.plan_hard_max_bounces`.
- F9, fixed as proposed: Scope In names the three pinned specs and what changes in each.
- F10, fixed as proposed: the declared PATTERN-shed-stop-chain text admits batten's override, at 280 runes, within the cap.
  "goes rule table, arbiter, then parent" became the equivalent "skips the tackler" to fit.
  The Constraints line matches; Notes for the plan writer has no line on it.
- F11, fixed as proposed: Scope Out's pawl bullet and D2's exception name D13's override move as the one pawl path this task changes.
- F12, fixed as proposed: D13 removes the three `arbiter*` `KeyCopies` entries and loom's `migrateShedOverrides`, and the acceptance row checks both.
- F13, fixed as proposed: D3's peers bullet takes the off-machine recipe as `boardRecipe` does, and the acceptance row adds an off-machine pawl peer.
- F14, fixed as proposed: Tapestry-Preflight maps to Preflight, so the `batcher.yaml` load runs again.
  Re-running Preflight is safe: `internal/preflightshed/preflight.go` reads no history.
  The acceptance row matches.
- F15, fixed as proposed: D2 places the non-loom refusal in the `lym loom` command path, not `loomcli.ArmAt`.
- F16, fixed as proposed: the first Open risks line is dropped.
  The support log's question-ledger line no longer says the point was raised under Open risks.
  Its D4 and D6 ledger lines name the new picks.

## Self-check

All four forks returned reports.

- fix-claims, BLOCKING, D4's fence claim is false against `fencedLines`. Merged with the fix-proposal MEDIUM below. Fixed as proposed: D4 makes the parser's fence rule CommonMark's, and the acceptance row adds an inner code block and an unclosed fence.
- fix-proposal, MEDIUM, a fenced body breaks on an inner code block or an odd fence count. Merged with the fix-claims BLOCKING above and fixed there.
- fix-claims, MEDIUM, D13 credits `ValidateLocal` with a way forward its retired-key message lacks. Fixed as proposed: D13 adds a check of its own, run before `ValidateLocal`, in `PrepareRunSettings` and in loom's and darn's local-layer loads.
- fix-way-forward, MEDIUM, a run resumed from Drift-Baseline without the goto keeps a baseline captured with no record. Fixed as proposed: Darn-Write's refusal names `goto --to Record-Write` then `resume` when the branch has no commit past the branch point, and the step route otherwise.
- fix-way-forward, LOW, the stop report must also pass the `verify` gate. Fixed with my own change: in place of requiring the report to pass the gates, Darn-Write reads the marker after a gate-failed Stuck as well as after a Done handoff. The gate outcome cannot be guaranteed on a base tree whose verify command fails.
- fix-consistency, LOW, the Auto-mode assumptions lack the new D4, D10 and D12 picks. Fixed as proposed.
- fix-consistency, LOW, D10's Bound list of routes to Preflight omits the retired-row map. Fixed as proposed.
- fix-consistency, LOW, D5 still says "darn writer row" after the build-segment entry. Fixed as proposed: it now says "Darn-Write row".
- fix-consistency, LOW, the PATTERN-refusal-way-forward constraint omits D6's halt. Fixed as proposed.

## Disputed

None.

## Deferred

None.
