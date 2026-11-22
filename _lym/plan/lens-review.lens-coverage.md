```yaml
lens: coverage
outcome: reported
```

Coverage lens: does every decision, criterion and branch have a card and a test row that would fail if it broke.

The plan never tests the wiring that applies autonomy at the real call sites. Its rows check the helpers (`WithRunAutonomy`, the setting-lookup wrapper, a stub `reset`) but not the places that call them.

**Findings**

1. **MEDIUM — D2 / card 4: no row shows a real halt notice carries the line.**
   - `TestWithRunAutonomy` tests only the helper.
   - No row asserts the lines posted at `internal/loomcli#loomCLI.goDriverDepsFor`, `internal/loomcli#newPrePushCheck` or `internal/battencli#notifyRunParent` carry `autonomy:`.
   - If any site skipped the helper, no test would fail. This leaves D2's "a halt notice of a run carries the run's autonomy" and D11's "a halt notice … carries that value" untested at the notice.
   - The card's ImpactSummary also says no existing test checks these sites' text, so nothing else catches it.

2. **MEDIUM — D2 / card 3: the warning on a failed read is not asserted, and two failure paths have no row.**
   - The criterion is "answers `false` with a warning when a stack in the chain fails to load".
   - The row "a beam file holding `autonomy: "yes"` reads false" asserts only `false`. No row checks the `logger.Warn` that names the worktree.
   - "A prime that does not resolve" and "a chain that does not read" are branches in the Intent that no row reaches.

3. **MEDIUM — D3 / card 7: forcing inside `wire` is untested.**
   - Two edits in `internal/loomcli#loomCLI.wire` have no row:
     - `ApplyAutonomy(loomCfg, …)`, which is what makes the Discussion lead spawn non-interactive.
     - The `autonomySettingLookup(configreg.SettingLookup(...), …)` wrapping, which is what makes a raddle proposal file without approval.
   - `TestAutonomySettingLookup` checks the wrapper over a stub. `TestRunAutonomy_ForcesTheRunsKeysOverARealHub` checks only `ParentReviewWanted` and `driftDeps`' `Config`.
   - If either `wire` edit were dropped, no row would fail. D3's criteria "the Discussion lead spawn[s] non-interactive with autonomous mode text" and "a raddle proposal files with no approval" go unreached.

4. **LOW — D3 / card 7: the tier1 row uses a different worktree than the criterion.**
   - The criterion says "with the parent worktree autonomous". The row sets `autonomy: true` in the pair's own `beam.yaml`.
   - A parent-only value is covered only indirectly, through card 3's lookup rows.

5. **MEDIUM — D3 / card 6: two outcomes of the new ended-ask branch have no row.**
   - The new check in `internal/driftengine#DriftVerify.judge` has two outcomes besides the halt:
     - Under `Autonomous`, a done ask stores the rulings.
     - Under `Autonomous`, a cleared ended ask does not halt.
   - The planned rows are only the three end states under autonomy plus the non-autonomous case.
   - The non-halting outcomes go unreached. So does `resumeJudgeSession` not clearing when `Autonomous` is false.

6. **LOW — D3 / card 6: a resumed halt may not reach `resumeJudgeSession`.**
   - The halt fires after `runAgent` returns a session. The card then claims "the judge session mark stays open, so the resume re-enters the same spawn".
   - No row checks the mark is still open after a halt that follows a finished `runAgent`.
   - If the mark closes, `resumeJudgeSession` is never reached and the clear never happens. The row "a second call after the halt … leaves the ask `Cleared`" would catch this only if it drives the real mark.

7. **MEDIUM — D4 / card 14: a rule from the decision record is missing, and the card's claim about guard sites is false.**
   - D4 says "Goto admits that only on a run halted at PR-Review or later, never on one PR-Review still holds."
   - `shedengine.Goto` has no direction check (`internal/shedengine/goto.go:17-69` shows only the recipe `Guard`). Card 14 swaps the rule for "driver not live" and adds no admission check or row.
   - The card also says `gotoGuardEndingFixers` is the builder "through which every site that moves a run onto a row builds its guard". In fact:
     - `internal/loomcli/arbiter.go:159` passes `c.settledGuard()` bare.
     - `internal/loomcli/drivewiring.go:216` does the same (`GotoGuard: gotoGuard`).
   - A goto through those paths would not clear the record.

8. **MEDIUM — D4 / card 14: the real `reset` closure and the wrapping are never run by a test.**
   - The real `reset` calls `ClearReviewSession` over `shedrun.PullRequestRecord` and then commits. The wrapping happens in `gotoGuardEndingFixers`.
   - `TestPRReviewResettingGuard` uses a stub `reset`.
   - The criterion "goto onto PR-Review clears the record's autonomy and session fields" has no row through `lym loom goto`.

9. **MEDIUM — D4 / card 10: no row reaches a failed ask.**
   - The Intent says "a failed ask is logged and the next entry asks again".
   - No `TestPRReview_Session` row scripts an `AskPRReview` error and asserts `ParentReviewAsked` stays unset.
   - An ask that ends no-parent, with no error, is unspecified: is `ParentReviewAsked` set or not?

10. **LOW — D4 / cards 12 and 13: the end-to-end criterion is checked only in parts.**
    - The criterion is "answer … recorded, and `lym loom pr-continue` then ends the review".
    - Card 13 stubs `standing`, so the real answer → `ReadPRReviewStanding` → `pr-continue` chain is checked only piece by piece (cards 9, 12, 13).
    - Two branches have no row: card 12's answer refusal for a PR-Review ask that is "not the latest and ended", and the `askPR` error path.

11. **LOW — D4 / card 10: the "judging stop" half of the toggle criterion has no row.**
    - Only the pause restart is a row. The criterion also names "a judging stop".

12. **LOW — D4 / card 13: the "row gates that head again" half of the mailed-head criterion is not asserted.**
    - The verb clears `MailedHead`, but no `TestPRReview_Wait` row checks the row then gates that head.

13. **MEDIUM — D6 / card 15: no row reaches a failing seam.**
    - The Intent says "a seam that fails is logged and decides nothing on that wake".
    - No `TestBeamAwait` row sets the `holdsCommits` or `workRemains` error fields that the card adds to `world`.

14. **MEDIUM — D9 / card 18: the start pointer has no row.**
    - The fresh-launch pointer in `internal/orchcli#orchCLI.launchOrch` is pinned at `start_test.go:1641-1650`, per the exploration, but no listed test is updated for it.
    - The criterion names "the start, resume and reload pointers".
    - Two edited pending-resume sites have no row:
      - `internal/orchengine#Watcher.startAutoReload`
      - `internal/orchengine#Watcher.reloadAfterCompaction`
    - Only `startClearing` (the handoff case) is covered.

15. **LOW — D1 / card 1: the card cites rows that don't pin the criterion.**
    - It says the row `a lower value overrides a parent's on` runs with `hub: "on"`. That row has no hub value (`internal/dobbyengine/tasksetting_test.go:72`).
    - The criterion "worktree false over default true" is in fact pinned by `the prime's own file applies when nothing nearer sets the key` (prime off, hub on, line 74). The card's evidence is wrong; the coverage is not.

16. **LOW — PATTERN-stencil-ownership / card 10: stencil sections move into Go strings.**
    - Card 10 moves the stencil's existing "Your parent's review" and "The operator's ending" sections into Go-chosen text.
    - That embeds prompt prose that today is read through `stencilstore`.
    - `mode_rules` is a precedent for Go-chosen mode text. Even so, moving whole existing sections out of the stencil should be a stated departure in `## Shared Decisions`.

17. **LOW — D8 / card 16: where the hook's `location` comes from is unstated.**
    - The exploration says the hook "does not know the worktree or anchor". `wireProduction` would need `resolveLocation(cwd)`.
    - If that resolution fails, `orchAutonomous` reads false and every reply is forwarded. No row reaches that case.

**Tool calls**
- I ran one grep over `internal/shedengine/goto.go` and the loomcli goto guard sites. My context had no goto direction rule and no full list of guard sites, which findings 7 and 8 needed.
