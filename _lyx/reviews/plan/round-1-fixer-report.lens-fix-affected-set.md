```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the set search for each set-premise finding (F1, F2, F4, F5, F11, F17).

Findings left unchecked: none of the set-premise findings. F12 and F16 are not set premises, so this lens did not rerun them.

**MEDIUM: card 25 misses `TestArmAt_RunOwnedElsewhereWritesNoSeed`.** This is F1's card-25 half.
- `internal/battencli/owner_integration_test.go:38` asserts `"lyx adopt " + slug`.
- Card 25 rewords `refuseStartOwnedElsewhere` to name `lyx adopt &lt;AdoptSubject&gt;`.
- Under card 3's fixture change for this test (the other machine recorded as owner through the prime), the subject becomes `root`, so the assertion breaks at card 25.
- The test is on card 3's Edit list but not on card 25's. The fixer's new card-25 sentence lists only `TestRefuseStartOwnedElsewhere`'s three rows.

**MEDIUM: card 3 misses `pairteardown#TestTeardown_OriginSeams`.** This is F1's card-3 half: a test that seeds `owner_machine` straight into the board.
- `internal/pairteardown/originseams_test.go:87-92` seeds `{"slug":"mine","kind":"task","owner_machine":"here"}` and `theirs` with `owner_machine:"other"`, under `prime_owner:"other"`. It asserts that the owner seam reads each entry's effective owner.
- Card 3's `Effective` ignores a task's recorded owner, so `mine` reads `other`, and the test breaks at card 3's gate.
- The test is only on card 25's Edit list. The fixer's added fixture sentence on card 3 names three tests and not this one.

**LOW: card 3 may miss the `owner_machine` subtest in `TestUpsertFieldAllowlist`.**
- `internal/boardengine/store_test.go:486-526` (`owner_machine_is_accepted_only_as_the_effective_owner`) writes `owner_machine` on task `t1` and accepts it when it equals the effective owner.
- Card 3 changes `checkOwnerPayload` and makes `normalizeTask` clear a task's owner. The refusal rows still match (`lyx adopt` plus `"neve"`), but an accepted row that checks the stored owner would break.
- The test is on card 1's Edit list, not card 3's. Not verified whether the accepted rows read the stored owner back.

**LOW: F17's stale-comment set misses `smoke_cycle_test.go`'s file header.**
- `internal/orchcli/smoke_cycle_test.go:6`'s file header says "because `lyx orch start` spawns a…".
- Card 10 now rewords `startdobbies_test.go`'s header but not this one, though its one test, `TestSmokeOrch_OneFullCycle`, is on card 10. Card 26 scans production files only.

Other members checked and covered:
- F2: `cancelDeps.Owner` is used at `cancelcmd.go:183`, `cancel_test.go:118` and `cancel.go:385`, all on card 25 now.
- F4: the callers `adopt.go:135`, `beamland.go:60` and `originreconcile.go:733,758` sit on cards 4, 13 and 25.
- F5: `adopt_test.go:304` (`TestAdopt`, card 4) and `TestDropLocal` (now on card 4).
- F11: `owner.go:151,201,269` and `resumereconcile.go:128`.
- The other `lyx adopt` + slug assertions are on card 25: `run_test.go` in `TestRunCmd_ResumeDispositions`, `wire_seams_test.go:440`, and `cancel_test.go:427,434`.
- `startguard_test.go:253` names the beam itself, which remains a valid adopt subject.
- The other test mentions of removed verbs are on cards 1, 10, 12, 25 and 26.

Tool calls:
1. grep for tests seeding a task owner or asserting a slug-named adopt: to rerun F1's search, which my context lacked.
2. Mapped those hit lines to their test functions: to know which test each assertion sits in.
3. Read the unlisted candidate sites (`store_test.go`, `startguard_test.go`, `originseams_test.go`, `owner_integration_test.go`): to judge whether each breaks.
4. grep for callers of the re-signed members, plus test files naming removed verbs: to rerun the F4 and F17 searches.
5. Mapped the F17 hits to their functions: to see which cards already hold them.
6. grep of the plan cards for the remaining candidate tests: to confirm card coverage.
