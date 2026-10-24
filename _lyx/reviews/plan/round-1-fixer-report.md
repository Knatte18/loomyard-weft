# Plan-Burler fixer report — round 1

Card numbers below are the final ones unless marked "old".
The review's cards 3 to 22 keep their numbers, old card 23 is gone and its lines are spread over four cards, old cards 24 to 32 are now 23 to 31, and old card 33 is merged into card 31.
`lyx loom validate-plan` is clean after every change.

## Changes

- **F1 (BLOCKING) — fixed differently.**
  I did not make the proposed move of old card 23 ahead of card 3.
  I made that move first, but the self-check measured it: card 23 saves 482 bytes, from 15651 to 15169.
  The four growth lines add 69, 122, 133 and 137 bytes, so the last growth card leaves `PATTERN.md` at 15630.
  That is above its HEAD, 15493, and above `HeadroomBytes`, 15564, so that card's `lint-pattern` step still fails (`internal/pattern/headroom.go:12`).
  With any card order, the card that leaves the file at its final 15630 must not grow it.
  So the nineteen shortened lines now ride on the four growth cards, each card cutting at least as much as it adds, verbatim from the recorded decisions:
  - card 3: 2 lines, −69 for +69;
  - card 7: 4 lines, −129 for +122;
  - card 19: 5 lines, −139 for +133;
  - card 31: 8 lines, −145 for +137.

  The final size, 15630, stays under the 15651 base, so `pattern-headroom` still passes.
  Old card 23 is removed.
  The new overview decision `pattern-cuts-ride-growth` records why.
- **F2 (BLOCKING) — fixed as proposed, with the affected sets taken from searches rather than the review's list, and one part fixed differently.**
  - Card 1:
    - adds `templateDefaults`, `TestCardGateCommand`, `TestRecoverSpawnOrAttach`, `TestBeginBatch_PromptFile` and `roundGateFixture.round`, the last reading `finalize` until card 4 deletes it;
    - the Intent names the search for `Gates`, `TierList(` and `gates:` seeds.
    - The proposed `TestVerifyGate_Scenario` is not added: it seeds no `gate.yaml`. The list-shape seed at `internal/loomshed/verifygate_integration_test.go:171-175` is `roundGateConfig`, which card 1 already lists and which `TestVerifyGate_RoundScenario` uses.
  - Card 6:
    - re-signs `withTreeScope` to `func withTreeScope(trees string) string`;
    - adds `docs/overview.md`'s loom-entry sentence;
    - deletes `treeScopeTargetsMarker`, `treeScopeControlOperators`, `treeScopeOffCommand` and `treeScopeVerifyCommand`, plus `treeScopeAddTreeCommand`. A search shows that last one is read only by the retired `ErrTreeScopeWithoutTrees` refusal (`internal/darnengine/config.go:194`).
  - Card 14:
    - adds `TestGateTest_Scopes`, whose guard-step subtest is reworded to the per-tier group, and `docs/overview.md`'s `guard_tier` mention;
    - the search for `guard_tier` found loomshed's seed is `roundGateConfig` (`guard_tier: 1`, line 146), not `TestVerifyGate_Scenario`, so the card now lists `roundGateConfig` in its place.
- **F3 (MEDIUM) — fixed as proposed.**
  - Cards 23 to 31 are `Edit` cards over the same package self glyphs, each with the proposed `ImpactSummary`.
  - The overview's `retag-rules` second sentence is rewritten.
  - Old card 33's Create, Edit, Uses, Intent and Verify are merged into card 31, and old card 33 is removed with `lyx loom remove-card`.
  - The Card Index line for card 31 names the scan.
  - The `scan-after-last-retag` decision is dropped.
- **F4 (MEDIUM) — fixed as proposed.**
  Card 4 re-signs `darnTreeScope` to `func(gateKey string) (string, bool, error)` and lists `TestDarnTreeScope_ClassifiesAtEachCall`.
  Card 14 re-signs `guardRunOf` to `[]tierRun`; its one caller is `runTest`, already listed.
- **F5 (MEDIUM) — fixed as proposed.**
  In card 21, only `TestNewDarn_Routing` asserts Describe's entry gate.
  `TestDarnRouting` asserts only the kept edges and the budget on the perch rows.
- **F6 (MEDIUM) — fixed as proposed.**
  Card 19's way forward for a value the local layer holds is `lyx config <module> --layer local --set <key>=<bool>` in the task worktree, since `run_settings` reach the pair only at Seed-Child.
- **F7 (LOW) — fixed as proposed.**
  - Card 19 adds the recipe-wide `Env.Resume`: `lyx batten run <slug>` for batten, `lyx loom resume` for loom and darn.
  - `loomCLI.wire`, `battenCLI.wire` and `envkit.FullEnv` fill it.
  - The evaluation error ends with it, and `TestBuild_EntryGate`'s non-bool rows assert it.
  - The ImpactSummary says two Env fields.
- **F8 (LOW) — fixed as proposed.**
  Card 1 names `lyx config reconcile --layer default --apply`.
- **F9 (LOW) — fixed as proposed.**
  - Card 23's Verify uses `/covers: *\[ *[0-9]/`, and the Intent says an empty `covers: []` passes.
  - `--resolved` renders lists in flow style.
  - A sample with `covers: []` and `covers: [0]` reports only the second.
- **F10 (LOW) — fixed as proposed.**
  - The follow-up upsert is of kind `task` with the type label `enhancement`, the one `gate-runs-once` carries on the board.
  - Its body adds card 13's `defaultRedundancyTags` and card 16's `isolationTags` and `isolationTagSets`.
  - This text now sits in card 31.
- **F11 (LOW) — fixed as proposed, with a fuller set.**
  - Card 5 drops the empty-`GateKey` default.
  - A search for `Deps{` and `verifyGate{` over the landing tests found more members than the review named:
    - `newMergeInFixture` and `newFixerRunFixture` set `merge-in`, and `newTestDeps` sets `finalize`;
    - `newGateFixture`'s `verifyGate` literal, which the kind-asserting tests use, now takes the row key (re-signed). Its callers are listed: `newFinalizeGateFixture`, `newFinalizeVerifyFixture`, `TestFinalize_MergeJudgement` and the `TestVerifyGate_` tests.
- **F12 (LOW) — fixed as proposed.**
  - Card 7's `TestGreenRecord` drops the changed-command and `-- -count=1` break rows, which are left to `TestGateTest_Scenario`.
  - It keeps the rows the scenario does not reach.
  - Per the self-check, it also keeps the positive exact-args carry under `-- -count=1`, which the scenario does not reach.

Renumbering: every prose card reference the verbs reported was corrected by hand after each `remove-card` and `insert-card`, including the overview's `Applies to` lines, and a final grep checked them.

## Self-check

- fix-proposal, BLOCKING (F1's move leaves the last growth card above the headroom line): fixed differently, by spreading the shortenings over the growth cards as described under F1. The fork's suggestion of new wording for a line is not taken, because the recorded decisions fix every wording.
- fix-claims, BLOCKING (the same headroom gap, and card 3's "stays under its base size"): duplicate of the fix-proposal finding, fixed by the same change. The moved card and its sentence are gone.
- fix-claims, MEDIUM (card 5's `TestVerifyGate_CheckMapsVerifyResult` builds no `Deps`): fixed as proposed. `newGateFixture` takes the row key.
- fix-affected-set, MEDIUM (the `verifyGate{}` literals in `newGateFixture` and `TestVerifyGate_Clean` are missed): duplicate of the fix-claims MEDIUM.
  - `newGateFixture` was fixed as proposed.
  - `TestVerifyGate_Clean` is listed only as a caller of the re-signed fixture. Its zero `verifyGate{}` calls only `clean`, which records no kind (`internal/landingshed/verifygate_test.go:449-453`), so it needs no key.
- fix-claims, LOW (card 7 dropped the exact-args positive carry): fixed as proposed. The row is restored to `TestGreenRecord`.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.
