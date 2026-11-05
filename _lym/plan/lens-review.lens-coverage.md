```yaml
lens: coverage
outcome: reported
```

Coverage lens: every decision, acceptance criterion and Intent branch of the drift-rework plan checked for a carrying card and a test row that would fail if it broke.

**Findings**

1. BLOCKING — D3 (`docs` profile), card 18. Card 18 adds `drift: {scope: off}` to the docs profile's `darn` block but does not list `internal/runprofile#TestTemplateDocsProfile`. That test pins `Settings("docs","darn")` with `reflect.DeepEqual` against `{darn, loom}` only, so it will fail. It is also the natural row for D3's "the `docs` profile's darn block sets `drift.scope: off`" criterion, which no card currently tests. Evidence: `internal/runprofile/runprofile_test.go:192-212`.

2. MEDIUM — D13, card 20. The verdict must list every open range "with its `why` and the judge's sentence". The `why` has no source here: card 20 renders from `ReadRecords` only, and card 7's `JudgedRange` holds case, range, checked, strand and at, with no `why`. The `why` lives in the sealed expectation and its amendments, which the verdict never reads. Card 20's "open ranges with `why` and sentence" row cannot pass as specified. Evidence: card 7 `JudgedRecord` bullet; card 20 Intent.

3. MEDIUM — D9, card 15. D9 says the judge sees "for an added case … the new file, for a removed case the old one". Card 15's `JudgeRequest` and `JudgeTable` values carry the cells, request path, judgement path and subject reading, but no path to an added case's current-capture file or a removed case's baseline file. No row covers it.

4. MEDIUM — D11/D12 admission range, card 19.
   - `Amend` is admitted at any blocked row "past Drift-Expect up to and including Drift-Recheck". `Defer` gets the same range when `CheckBaseline` fails.
   - In tapestry that range also takes in Parent-Align, Code-Bouncer, Code-Burler-Review and Code-Burler-Fix. D11 admits amend only at a Stuck writer, Drift-Verify or Drift-Recheck.
   - No row shows a blocked Parent-Align or Code-Review row being refused.

5. MEDIUM — D2 (`lym drift status` says the compare is ignored under `lines`). No card carries this. Card 22 edits `statusCmd` and `readStatus` without it.

6. MEDIUM — D6/D7/D8 "reports without failing", card 10. `NewDriftGate`'s pass branches on `BaselineFault`, `RecordsFault` and `SealFault` have no row. `TestCheck` asserts these faults only at the `Check` report level, and its gate rows cover only an unexpected cell, scope `off` and a deferral. Also untested:
   - the seal check under `command` mode (D6 criterion);
   - `lym drift check` reporting a missing baseline without failing (D8 criterion). Card 12's rows cover only an unexpected value and the before-Drift-Expect case.

7. MEDIUM — D7/D12 at Drift-Verify, card 16. Card 16 Intent step 3 has two branches with no `TestDriftVerify` row:
   - no counted capture is a Stuck naming `lym drift defer`;
   - an edited baseline file or missing baseline directory before the judge is a Stuck naming `lym drift defer`.
   Only the edit made during the judge session has a row. These are D7's "an edited baseline file stops Drift-Verify naming `lym drift defer`" and D12's "baseline directory is gone stops … Drift-Verify".

8. MEDIUM — D5/D9 exhausted entries, cards 13 and 16. An exhausted `expectation` entry is a Stuck whose resume respawns the expect agent, and likewise for `judgement`. No row reaches either. `NewExpectationGate` has no direct row: no missing-file or parse-failure row, unlike card 15's `TestJudgementGate`.

9. MEDIUM — the prefixes Stop-prefixes depends on, cards 13, 16 and 19. `Defer` admits a halt only by `DeferHaltPrefix`, and parent routing relies on `DecisionHaltPrefix`. No `TestDriftVerify` or `TestDriftExpect` row asserts the produced Stuck reasons carry these prefixes. `TestDeferWaive` builds its statuses by hand with `statusAt`, so a real capture-failure Stuck that lacks the prefix would be refused by `defer` with every row still passing.

10. MEDIUM — D7 tapestry route, cards 10, 11 and 16 (Shared Decision `tapestry-fix-route`).
    - D7 says the route "must be reachable from a Stuck Drift-Verify and a Stuck Drift-Recheck", and its own rationale warns that a re-entered Webster has no card to run.
    - The plan assumes a goto to Webster re-spawns Merriam and runs the run-level `drift` entry with every card done. No card researches or tests this; I found no websterengine code confirming it.
    - `FixRoute`'s tapestry string has no row either; only darn's has one.

11. MEDIUM — D3 records written before this lands, cards 10 and 16. A scope record with no `Diff` should record config's `diff` and `key_patterns` at its next drift row and keep them, and the gate and check should read config until then. No row covers either: card 10 tests only "recorded diff wins". A later `key_patterns` change not altering identities has no row.

12. MEDIUM — D3 old `partial` record, card 10. `Check` leaving out the skip list's cases when the scope record says `partial` has no `TestCheck` or `TestDriftVerify` row. Only `DiffCaptures`' `leaveOut` (card 3) and the verdict (card 20) are tested.

13. MEDIUM — D7 baseline digest on the seal, card 13. "A capture counted before this lands, with no digest, gets one on Drift-Expect's seal commit" is in the Intent (the `Drift-Baseline` trailer when `Unguarded`), but `TestDriftExpect` has no row for it.

14. MEDIUM — D4 and D6 in `command` mode, cards 16 and 17. Two branches have no row:
    - a check whose judge ruled every case `matches` counting as a pass for the unchanged-head skip;
    - card 16 step 6, recording `Unsealed` at the first Drift-Verify past Drift-Expect with no seal.

15. MEDIUM — the `amend` verb, card 19. These refusal branches are unreached (`PATTERN-refusal-way-forward`):
    - `--restore` when nothing failed;
    - a flag set holding other than exactly one of `--entries`, `--restore` or `--discard`;
    - an unknown slug in `openSlugWorktree`;
    - the `--entries <file>` read.
    `amendCmd` has no test at all. Amend entries under `command` mode (validated by D2's rules, dropping case rulings) have no row.

16. MEDIUM — D7 slot criterion, card 9. The criterion is "a check inside a gate and a fork's check with one free slot both finish". The planned row covers only the inheriting check alone.

17. LOW — D4 vs card 16 and the Shared Decision `old-chairs-end-through-the-strand-table`. D4 names `runproc.EndRole`, and the plan swaps in a strand-table stop recorded only in Shared Decisions, not through the parent's `lym loom decision add`.

18. LOW — branches with no row:
    - card 1: `FindTrailerCommit` on an unborn weft; the trailer-key charset refusal.
    - card 3: `CompileKeyPatterns` with no `key` group.
    - card 9: a current capture that exits 0 and writes no file.
    - card 10: no row asserts that the tapestry Webster row's `drift` entry resolves (`TestWebsterEntry_GateKeys` is untouched).
    - card 13: a closed mark with an invalid `expected.yaml`.
    - card 16: a judge resumed with the same spawn id.
    - card 17: an amendment since stored rulings, which forces a re-judge.
    - card 20: a `command`-mode verdict line.
    - D10: an open range replaced by a later listing.

19. LOW, outside this lens — `internal/reportengine/timingspans.go:102,120` names `DriftRecheck`. Card 23 deletes the engine without listing it.

**Tool calls**
- Grepped websterengine for re-entry with every card done and listed runprofile tests: to check D7's tapestry route and whether a test pins the docs profile.
- Read `TestTemplateDocsProfile` and grepped `runlevel.go` for pending cards: to confirm the exact-equality pin and the Webster re-entry behavior.
- Grepped websterengine and loomshed for done-card handling: to look for a Merriam re-entry path once every card is done.
