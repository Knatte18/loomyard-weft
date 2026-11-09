```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F6, F8, F9, F7 and F1, and check that the fix covers every member.

**Unchecked findings:** F2, F3, F4, F5, F10 and F11. Their premises are not a set of sites under one key, so there was no search to rerun.

**MEDIUM: F1's set of redundant tests still has a member.**
- Changed text: card 7's removal of `TestJudgeStanding` and card 8's removal of `TestRunsInOrch`.
- Summary: card 6's `TestPauseEntries` tests the unexported `pauseEntries`, and most of its rows repeat rows other planned tests already pin. F1 cites `PATTERN-test-speed` against exactly this.
- Evidence:
  - The owner entry converted, the other caller's entry kept and the beam's owner entry converted are each also a `TestAdopt` row in card 6 ("an `owner` entry turned to `adopt`; another caller's entry kept…", "A beam row: `alpha`'s own-scope `owner` entry is converted").
  - The listed-slug skip is also card 7's `TestRunWorker` row ("A record listing a slug in `Inputs.Paused` whose entry the operator lifted since… leaves that slug with no entry").
  - Card 6 says the two overlap: "`TestRunWorker` (card 7) carries it, and `TestPauseEntries`' listed-slug row pins the skip itself".
  - Only the rows "no entry" and "beam with no entry left" may pin something no other test does.

**LOW: F8 leaves a stale comment over the run constants.**
- Changed text: card 6's new Delete entry `internal/adoptcli#runningHereRun` and its sentence "`runningHereRun` goes with `holdTargets`' live-pair exception".
- Summary: the comment over that const block still describes the held and running-here outcomes. Card 6 deletes `runningHereRun` and renames `heldRun` but never says the comment is reworded.
- Evidence: `internal/adoptcli/adopt.go:36-37` reads "A run is owned elsewhere after the owner write, held for its subject's adopt check, or running here on a live pair."

Members checked and covered:
- **F6:** every hit of the green-end and holds texts is covered, either by the new edits (`fixloop.go:335,340`, `battencli/owner.go:106`, `passes.go:108`, `docs/overview.md:785,787`) or by existing card bullets:
  - cards 1, 4 and 10 already list `check.go:502`, `finalize_owner.go:130`, `beamcli/doc.go:74`, `beamcli/cli.go:780`, `battenshed/doc.go:98`, `overview.md:1570`, `pauserecord` and `dobbyengine/doc.go:158`, the adoptcli `doc.go` and `PATTERN`;
  - cards 6 and 8 cover the `cli.go:39,41` Long text.
  - `check.go:495` still holds, since the green end remains.
- **F8:** `callerLabel` (`deps.go:91`, `green.go:253`), `PauseLifted` (`adopt_test.go:521`, the integration test at line 115) and `runningHereRun` (`holds.go:175`, `adopt_test.go:604`) all sit in targets that cards 4 and 6 delete or rewrite. `runreport.go`'s `runStateRunningHere` is a different symbol.
- **F9:** the only descriptions of `AdoptScratchDir` are `junctionnames.go:188` and `fabricengine/doc.go:1122`, both now on card 3. The adoptcli `doc.go:270,423` mentions belong to card 10.
- **F7:** the test's assertions at lines 112–191 are each accounted for by the changes to cards 6 and 7: the `Held` flag, the caller `HoldCaller`, `holds.json`, `PauseLifted`, the adopt back's `t.Fatalf`, the `--drop-local` refusal and the `--keep-local` owner. `AdoptAs` is in `export_test.go`, which card 7 lists under Edit.

Calls made:
1. A repository grep for the green-end and holds texts. My context held only the spans the review cited.
2. A grep for `PauseLifted`, `runningHereRun`, `callerLabel` and "running here". My context lacked their uses outside adoptcli.
3. `adopt.go:30-45` and a grep for `AdoptScratchDir`. My context lacked the const-block comment and the other descriptions of the directory.
4. `adopt_integration_test.go:100-195` and `AdoptAs`. My context had only grep lines for that test.
