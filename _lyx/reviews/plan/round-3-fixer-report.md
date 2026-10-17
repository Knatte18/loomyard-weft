# Fixer report — plan, round 3

## Changes

- **F1 (fixed as proposed, then corrected by the self-check):**
  - `05-wire-segments-and-setting.md`: the `TestWire_ReviewKeysReachTheEnv` bullet now asserts `Segments` for both wirings.
    For `fanned`, that covers Discussion-Review's `ReviewKey` `discussion_reviewer_model` with its `Models`, and Webster-Review's `FixKey` `webster_fixer_model` with its `Models`.
    The `Setting` read moves to `fanned.env.Setting("loom", "plan_reviewer_fan")`, which answers `"full"`.
  - `08-retire-row-maps.md`: names `fanned.env.Setting` as the loom read it keeps.
    It also says `fanned`'s per-row fan, `RowModelKeys` and `RowReviewModels` expectations go, while its `Segments` assertions stay.
- **F2 (fixed differently):** `06-review-rows-read-segments.md`'s Settings paragraph drops the claim that a lookup error "names the template that lacks the key".
  - It adds that card 1's unregistered-module and unknown-key errors carry their own `way forward`.
  - It adds that an error propagated from `configengine.Load` or the decode stays as it is, because the wiring's strict load of the same stack has already passed for `loom` and `darn`.
  - Not made: the proposal's label "transient refusal".
    `contracts/specs/refusal-spec.md:31` defines transient as a condition that clears by itself or by a named action, and its message names the retry.
    An invalid-YAML or guard-key `Load` error does neither, so the sentence rests on unreachability instead.
  - The test sentence about `fakeSetting`'s text carrying no way forward is unchanged.
- **Self-check additions:**
  - `05-wire-segments-and-setting.md`: `TestWire_DarnRecipe`'s "set fans and models" case asserts that `c.env.Setting("darn", "darn_fixer_fan")` answers `"fix-code"`.
  - `08-retire-row-maps.md`: the darn `Setting` read card 8 keeps now points at that case.

## Self-check

- fix-consistency, MEDIUM: card 8 said "only" the per-row fan and `RowModelKeys` expectations go, which left `fanned`'s `RowReviewModels` assertions (`wiring_test.go:1029-1035`) on deleted fields. Fixed as proposed.
- fix-consistency, LOW: `_lyx/discussion/decision-record.md:117` (D6) still says the way forward names "the template that lacks the key". Deferred: the file is outside the write surface.
- fix-proposal, MEDIUM: the same `RowReviewModels` gap as the fix-consistency MEDIUM. Merged with it and fixed there.
- fix-proposal, MEDIUM: "transient refusal" misapplies `refusal-spec.md:31`. Fixed as proposed.
- fix-claims, MEDIUM: the same "transient" label. Merged with the fix-proposal finding.
- fix-affected-set, LOW: card 8 kept a darn `Setting` read that no card added, and named no seed case. Fixed as proposed.

## Disputed

None.

## Deferred

- The self-check's LOW on `_lyx/discussion/decision-record.md:117` (D6): its sentence that the way forward names "the recipe key to fix or the template that lacks the key" is now stale against card 6.
  The file is outside this fixer's write surface (`_lyx/plan`).
