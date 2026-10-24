# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING), fixed as proposed: D11 now has `lyx gate test` keep a per-package launch list in the run record for each go-json tier run.
  Each entry holds the gap from `start` to the next event and the package's elapsed time; packages from the cache or without test files are left out.
  The counter decodes `Package`, `Time` and `Elapsed` beside `Action`, and the rendered log stays output only.
  Records from before this task show no launch time.
  The rationale now names what records hold today (wait start, start, end, counts) and what the task adds.
  The acceptance criterion reads launch time from the launch lists, and a new criterion covers the list's contents.
  The OSL-1046 assumption now grounds the before side on slot wait and wall time, and puts launch share after the deploy only.
- F2 (MEDIUM), fixed as proposed: D13's Verify line also fails while any tier in `lyx config gate --resolved` carries a `covers` entry, and D13 says why.
  Constraints bullet 1 and Open risks bullet 2 name the `covers` case.
- F3 (MEDIUM), fixed as proposed: D8 states that a `pause_after` on a skipped row fires as for a Done.
  The run pauses at the routed row with `paused after <row>, as requested` and both conditions clear.
  A matching acceptance criterion is added.
- F4 (MEDIUM), fixed as proposed: D5 extends `namesGateOnlyTier` and the shell guard's raw go row to `tier2` and `tier3`.
  D13's follow-up removes the old spellings there.
  A matching acceptance criterion is added.
- F5 (MEDIUM), fixed as proposed: D9 has the build check a `module.key` operand against the config registry's templates, the known-key check `configreg.SettingLookup` makes, never through `Env.Setting`.
  `Env.Setting` serves the evaluation.
  A criterion says a build needs no task worktree.
- F6 (MEDIUM), fixed as proposed: D14's migration fires only while the hub file holds `tree_scope_verify`.
  Held empty, it empties `tree_scope` and prunes the key; held set, it prunes the key and keeps the trees; absent, it changes nothing.
  D14 also states the turn-on case.
  The acceptance criterion covers each case and a second reconcile.
- F7 (MEDIUM), fixed as proposed: D5 now holds the card order.
  D6 and D7 land before or with the first retag card, and the one-tag scan lands with the last.
  The scan is dropped from the Notes bullet.
- F8 (MEDIUM), fixed differently: the proposed "the guard step writes no green entry" was made, then replaced after the self-check.
  Writing nothing makes every repeat of a `diff` key re-run its guard groups, against the Goal's "a repeat on the same tree runs nothing".
  D4 instead has the guard step write its entries under its own `-run` args: an exact-args match carries a repeat, and such an entry never covers an empty-args request, which resolves the premise.
  D6 and the acceptance criterion say so.
- F9 (LOW), fixed as proposed: D8 states that every routed persist and every skip clears the entered row.
  A criterion covers a Stuck routed back through a skipped row.
- F10 (LOW), fixed as proposed (restrict option): a bare `module.key` reads a bool and a bare `budget.<Segment>` reads a number, and any other type is an evaluation error.
  The criterion names `no` and `1`.
- F11 (LOW), fixed as proposed: `cmd/testtiming` is dropped from D5's package-load list.
  D5 states its default runs `-tags tier0`, `-full` maps to `tier1`, and its spawn scan reads the new tags beside the old.
  D13's follow-up names the spawn scan.
- F12 (LOW), fixed as proposed: D4's Bound names ignored files, such as `.dev-bin`, as outside the clean check and the fingerprint.
- F13 (LOW), fixed differently: the proposal named "Webster's end gate in loom and the landing call in darn".
  The self-check found the darn half false: `internal/loomshed/verifygate.go:87-92` sets the darn form's lint base from the merge base and never reads a recorded pass, as D3 itself says.
  The rationale now names only Webster's end gate as giving loom's fix round its lint base.
- F14 (LOW), fixed differently: the sentence moved from Notes into D1 as proposed.
  The proposal's "reads as the key of the row that wrote it" was changed to "the row that reads it".
  `LandingFailure` names no row (`internal/verifytree/failure.go`), and only Merge-In, Finalize and their merge fixer read it (`internal/landingshed`).
- Support log: the Turn 6 entry is updated to match D11's new source.

## Self-check

- fix-claims: no findings.
- fix-way-forward: no findings.
- fix-consistency, MEDIUM: D3's darn clause contradicts D3's own darn lint rule. Fixed (see F13).
- fix-consistency, LOW: the Constraints' tolerant-records bullet omitted the run record's new fields. Fixed: the bullet now names the gate key and the launch list.
- fix-consistency, LOW: support log Turn 6 was stale against D11. Fixed.
- fix-consistency, LOW: D9's build check named no path into `shedrecipe` and `shedbuild`, which do not import `configreg`, and the setting-lookup Constraint did not mention it. Fixed: the caller tells the check as a recipe-wide `Env` function, and the Constraint names it.
- fix-proposal, MEDIUM: a guard step writing no entry re-runs on every repeat. Fixed with my own change (see F8).
- fix-proposal, MEDIUM: the landing record names no row that wrote it. Fixed (see F14).
- fix-proposal, MEDIUM: D3's darn half is false. Duplicate of the consistency MEDIUM, fixed.
- fix-proposal, LOW: packages without test files entered the launch list. Fixed: D11 and its criterion leave them out.
- fix-proposal, LOW: test files added before the last retag card could land untagged. Fixed: D5 has every card from the first retag on tag what it adds.

## Disputed

None.

## Deferred

None.
