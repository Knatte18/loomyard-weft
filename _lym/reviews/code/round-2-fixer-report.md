# Fixer-report — code review round 2

## Changes

- **F1 (BLOCKING), fixed as proposed, with one variation.**
  - `internal/burlerengine`: `ReviewChairRole(rolePrefix)` is the one exported helper that forms the chair's strand role.
    It rests on `ledRole`, which `Profile.rolePrefix` now uses too, and `reviewTable` builds its table prefix through it.
  - `internal/loomcli/wiring.go`: `segmentAssistBounds` resolves one keep-warm cap and one `loomengine.CapAssistTimeout` per review segment, under the chair name `agentname.Format(shortname, slug, ReviewChairRole(<segment's role prefix>))`.
    `loomSegmentSettings` takes the per-perch `assistBounds` map in place of the two single values.
  - Variation, the segment's role prefix: the review asked for a constant declared once.
    The prefix is `string(perch)`, from the `loomengine.Perch` constants, which are declared once and already equal the recipe rows' `role_prefix` values.
    No second constant was added.
    A new test, `TestReviewSegmentRolePrefixesNameTheirPerch` in `internal/loomrecipe/bouncerrows_test.go`, pins that every review row of the shipped tapestry and darn recipes sets `role_prefix` to its segment's perch name, so a recipe edit cannot silently break the link.
  - Test: `TestWire_ReviewKeysReachTheEnv` gains a fixture whose claude.yaml sets `keep_warm_cap_roles` for `code-burler-review` and the bare `burler-review`.
    Only Code-Review's assistant bound follows the code entry, and the other two segments keep the global cap.
    With the old single bare-role resolution the fixture fails.
- **F2 (LOW), fixed as proposed.**
  `internal/loomengine/config.go`: each timeout knob carries a `zeroMeans`, and the refusal message and the comment above the loop state it per key.
  `assist_reviewer_timeout_min` says "take the whole room left before review_timeout_min", the other three "defer to shuttle's run_timeout_min".
  The "must not be negative" fragment is kept.

Commits: `21a2deae` (F1), `43e3a97a` (F2) and the F1 role-prefix pin test, committed after the self-check.
Gate findings repair: two comments the round added were fixed-column-wrapped, in `internal/loomengine/config.go` and `internal/loomrecipe/bouncerrows_test.go`.
Each is now one sentence per line, committed as `fix gate findings`, and tier0 over `./internal/loomengine ./internal/loomrecipe` passes with no failures.
Verify gate failure, no change made: the full verify run failed tier1 on `internal/creelcli` `TestCreelCLI/a_waiting_verb's_envelope_that_cannot_be_written_keeps_its_exit_code_and_goes_to_stderr`, and its log held no failure detail.
This round's diff touches `burlerengine`, `loomcli`, `loomengine` and `loomrecipe` only, and none of them is imported by that test's code path.
Rerunning `lym gate test ./internal/creelcli --tier 1` passed, with 62 passed, 0 failed and 7 skipped.
The test starts a creel daemon and drives it under a clock and a 10-second poll, so it reads as a load-dependent flake outside this round's write surface, left as it is.
Gate: `lym gate test` over `./internal/burlerengine ./internal/loomcli ./internal/loomengine` and over `./internal/loomrecipe`, tier0, no failures.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, MEDIUM: the `role_prefix` link between recipes and perch names had no test, and the declared-once constant was not met. Fixed with the pin test above.

## Disputed

None.

## Deferred

None.
