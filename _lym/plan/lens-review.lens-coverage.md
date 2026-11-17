```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every D6 decision, every acceptance criterion and every branch the cards add against the planned test rows. I made no tool calls; all evidence is from the plan files and the code already read.

**BLOCKING**
- **D6 step 2 and the criterion "a plain `lym adopt` re-run under this machine's own `adopt-fix` hold leaves the fix task's branches and processes untouched".** Card 9's `movedRun` cannot pass its own row `AReRunUnderItsOwnAdoptFixHoldKeepsKindTaskTriesAndFilesNoSecondTask` ("no `TakeOver` of the fix task's run").
  - `movedRun` treats a run as moved unless its subject's `PriorOwner` in the hold reads as this machine.
  - After a takeover, the hold's `PriorOwner` is the old owner, per card 2 step 1 ("`PriorOwner` is the subject's effective owner before this write"). The fix task's run sits under that subject, so the rule reads it as moved.
  - `TakeOverRun` would then archive and reset its branches. The same happens to every run the first adopt already took, whose origin records name this machine's own takeover.
  - The rule needs another arm: a run whose origin record holds this machine's takeover entry is not moved, and neither is a run filed under this machine's hold.
  - Evidence: `09-adopt-foreground.md` Intent step 2 and its row list; `02-board-adopt-write.md` Intent item 1.

**MEDIUM**
- **Criterion "a green adopt check leaves no `bolt-hold` pause entry on the adopting machine when the adopt returns".**
  - Only the fake `DropBoltHold` call is asserted (`TestRunHoldCheck` row `AllGreen…DropsBoltHoldEntries…`).
  - The real filling in `hubHoldCheckDeps` is untested: `tenderengine.SyncBoltHold` with `Standing` false over the scopes of the live `BoltHoldCaller` entries. No row shows the pause record left with no `bolt-hold` entry.
  - Evidence: `08-adopt-check-ends-on-hold.md` Intent paragraph on `hubHoldCheckDeps`, and its row list.
- **D6 `--take-hold` "under `adopt-fix` the fix task stands and a failing check files no second task".** No row combines `--take-hold` with an `adopt-fix` hold and a failing check.
  - Card 9's `TakeHoldRunsEveryStepUnderTheRewrittenHold` and card 2's `TakeHoldRewrites…` stop at the rewrite.
  - Card 8's `AnOpenOwnFixTaskFilesNoSecondTask` uses this machine's own task, judged through the local run record (`judgeOwnFix`). On a takeover machine that record may not exist yet, or may read halted.
  - Evidence: card 8 step 4; `internal/adoptcli#judgeOwnFix`.
- **Card 9 `holdStands`, the "hold is gone" exit.** Only the "names another machine" exit has a row (`AHoldTakenOverByAnotherMachineStops…`). Nothing reaches a hold cleared mid-adopt. Evidence: card 9 Intent step 2.
- **Card 9 `validateRequest`'s new refusal of `--take-hold` with `--all`.** No row reaches it. The kept refusal rows predate it, and only `TestRunArchives` gets a `--take-hold` row. Evidence: card 9 Intent paragraph on `request`, and its row list.
- **Card 9 step 4, the catch-up.**
  - What a failing `CatchUp` does (stop or warn) is not specified, and no row reaches it.
  - No row asserts that the catch-up runs over the hub and pushes the owner's owed transitions without a reset. That is the second half of the "re-run under its own `adopt-fix` hold" criterion.
  - Evidence: card 9 steps 4 and 2 ("step 4 pushes its owed commits").
- **Card 9 step 6, the workspace.** The failure-is-a-warning branch has no row.
- **Card 9, a nested beam under an adopted root that keeps its own owner on another machine.** It is "reported as owned elsewhere and not taken over". The existing `TestAdopt` row `NestedBeamOwnedElsewhereUnderAdoptedRootIsReportedOwnedElsewhere` (`internal/adoptcli/adopt_test.go:834`) is not among the rows the card says it keeps or adds.
- **Card 3, the exhausted push retry.** The error naming `lym adopt <subject>` again has no row. Only the one-retry success row exists. Evidence: card 3 Intent step 5.
- **Card 3 `OriginRunStatus`.** This new exported reader has no test row. Neither found-false branch (no upstream, no record at the tip) is reached, and card 9 drives it only through a fake. The moved-run judgement depends on it.
- **Card 1, rewritten holds replayed.** No row covers the "rewritten holds replaced, matched by machine and subjects" branch.
  - This is the replay path of the `--take-hold` rewrite, whose machine changes, so a match "by machine" is ambiguous there.
  - It is also the replay path of `FixHold` and `ClearHold`.
  - Evidence: card 1 Intent, "Holds" bullet, and its row list.
- **Card 2 `WriteAdopt`, a rejected push that meets no new hold and no owner change.** The record's "otherwise it replays and pushes" end-to-end has no row. Only the losing race (`TwoClones…`) is covered.

**LOW**
- D6 step 8 says the dobby's notice at the limit names the failing tests and the fix task. Card 8 posts the line through `orchcli.NotifyFrom` with the check's own `ADOPT` program, not the dobby. Evidence: card 8 step 4 and the `hubHoldCheckDeps` `Notice` seam.
- Card 2: `FixHold`'s refusal over another machine's hold has no row; only `FixHoldWithNoOwnHoldRefuses` exists.
- Card 3: `DropHandOverMark`'s absent-mark branch has no row, and neither does `TakeOverRun`'s propagated anchor-change refusal.
- Card 4: no row for the `--take-hold=value` form, which the pattern comment claims to match.
- Card 5: the `UnderOwnHold` branch where the board or identity does not read (counts as no hold, logged once per episode) has no row.
- Card 7: the error return of `HubRunSlugs` has no row.
- Card 8: the `PullBolt` error in `runHoldCheck` step 1 has no row.
- Card 9: `AFailingCheckExitsNonZeroNamingTheFixTask` is not stated to assert that `BeamUp` runs after the failing check, which the record requires before the non-zero exit.
- Card 9: the `--take-hold` with `--all` refusal is text an agent may read. The card does not say it avoids spelling the flag, which `PATTERN-refusal-way-forward`'s constraint requires (no refusal an agent reads names `lym adopt --take-hold`).

Tool calls: none.
