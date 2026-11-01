```yaml
lens: coverage
outcome: reported
```

Coverage lens over the machine-switch plan: each decision, acceptance criterion and new branch mapped to its card and test row. Gaps only.

**MEDIUM**

1. **D4/D6, holds and tries files survive the check record's writes.** The criteria say "the check's own record writes leave the holds file's list in place" and "the count stands in the subject's tries file through a check's start and end, which rewrite the check record". No row runs `runCheck` with `holds.json` and `fix-tries.json` already in its directory and asserts both are unchanged afterwards. `TestHoldStore` and `TestGreenEnd` touch only the stores, and `TestRunCheck`'s new rows on cards 10, 12 and 13 assert notices and seam calls only. Evidence: the D4 and D6 acceptance criteria; card 11 `TestHoldStore`; card 12 and card 13 `TestRunCheck` bullets.

2. **D5, a reconcile failure is a no-verdict end that calls neither `Green` nor `Failed` and names `lym adopt <subject>`.** Card 13 changes that notice and states the branch. Its only planned row is the gate that exits before any tier. The existing row at `internal/adoptcli/check_test.go:207-223` asserts `"lym gate test --gate adopt"` in `record.Error`. The card does not say that row is reworded, and it plans no assertion that a reconcile failure calls neither seam. Evidence: card 13 Intent and Tests; `check_test.go:216`.

3. **D5, a green end lifts only its own subject's holds.** No `TestGreenEnd` row has a second subject's holds file or `HoldCaller` entry and asserts it stays. Evidence: D5 acceptance "a green check lifts only its subject's holds"; card 12 Tests.

4. **Card 12, failure branches with no row.** Step 1's failed lift (naming `lym tender resume <slug>`), step 2's failed `BeamUp` (naming `lym beam start <subject>`), step 4's skip of a child whose effective owner is another machine, and the `hubGreenDeps` build-error line are all unreached. Evidence: card 12 Intent steps 1, 2 and 4; its `TestGreenEnd` list.

5. **D2, the warning covers only one state.** The single planned row is "a rewritten branch of the prime". Nothing reaches `diverged` or `ahead`, a branch of an adopted beam's pair, the "no stop of that slug already reported" exclusion, or the `sharedStopText` exclusion. The criterion "a run pair, a beam pair and the prime pair behind origin are fast-forwarded by the reconcile step" has no row on any card and is argued only from the existing reconcile. Evidence: card 11 `reconcileWarnings` Intent and Tests; the D2 acceptance criteria.

6. **D7, a branch card 14 adds has no row.** `held_failed_check` requires that "this call started no check of the subject", so a failed or no-verdict record plus a check started by this call reads `waiting_for_check`. No row reaches that. There is also no row for `held_failed_check` from a running record whose process is gone (ended without a verdict), and none for the read-failure branch that leaves `runs` empty and adds a `warnings` line. Evidence: card 14 Intent and the `TestRunReports` list.

7. **Card 6, start-hold branches with no row.** Nothing reaches `max_runs: 0` still starting no front task, a covering pause or the start backoff holding a front task, or a `SettleFronts` error being logged without stopping the tick. Evidence: card 6 Intent and the `TestStartReady_FrontTask` list.

**LOW**

8. **Card 3, payload branches with no row.** Nothing reaches a `front` that is not a bool, an `on_land` that is not a string, or a new entry's `"front":false` / `"on_land":""` being dropped rather than refused. Evidence: card 3 `checkFrontPayload` Intent; `TestFrontTask` list.

9. **D8, "a halted or removed front task runs none".** For a removal through `fronttask.Remove`, no row calls `Settle` afterwards to show the dropped record runs no action. `TestRemove` asserts only that the record and entry are gone. Evidence: the D8 acceptance criterion; card 5 `TestRemove` and `TestSettle`.

10. **Card 13, ordering of the halted removal against the try limit and another open front.** The card removes a draft or halted own fix task (case 2) before checking another open front (case 3) and the try limit (case 4). D6 places the removal "before the filing", and in cases 3 and 4 no filing happens. No row pins the outcome when a halted own task meets either case. Evidence: card 13 `failedEnd` steps 2–4; `decision-record.md` D6 bullets on removal and the limit.

11. **Card 13, operand of "halted".** `fixRunHalted` reads the batten run record. D6 states the halted test on "its entry's run status having the state `blocked` or `failed`", while D7 uses the batten run record. The card does not say why the two agree. Evidence: card 13 `fixRunHalted`; `decision-record.md` D6 line 124 and D7 line 151.

12. **D3, a hand `lym batten cancel <slug>` lifts a `HoldCaller` entry.** No row covers this specifically. The plan relies on `liftSlugPauses` lifting every caller, which is plausible but untested for the new caller. Evidence: the D3 acceptance criterion; card 12 Bound.

Tool calls:
- Grepped `EscalationPending`, `arbiterstop.EscalationEligible`/`NewReaders`, `shedrun.RunDir`, the `OriginReport` `Prime` field and `TestRunCheck`'s rows, to check card 13's and card 11's operands and the existing rows; my context did not hold these.
- Read `lymdirs.LymDirName` and the reconcile-failure row at `check_test.go:207-223`, to check that run records sit under `_lym` and what the existing no-verdict row asserts; my context did not hold either.
