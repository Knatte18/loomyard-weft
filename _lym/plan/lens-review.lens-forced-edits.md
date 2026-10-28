```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens report on the four cards.

- LOW — Card 2 (D3), `Finalize.gateAndFix`'s doc comment: the comment at `internal/landingshed/finalize.go:640` says "An escalate report, an unusable turn and an attempt record that does not decode each halt with LandingStopPrefix". That is wrong once Card 2 gives a loom or darn escalate `EscalateStopPrefix`. Card 2 lists `Finalize.runFixer` but not `internal/landingshed#Finalize.gateAndFix`, which spans that comment. Card 1's whole-file `finalize.go` target runs before Card 2, so it cannot carry the change. Fix: add `internal/landingshed#Finalize.gateAndFix` to Card 2's `**Edit:**`.
- LOW — Card 2 (D3), the Parent-Align doc comment: `internal/landingshed/parentalign.go:121` says every halt of the alignment is a halting Stuck "whose reason opens with LandingStopPrefix". The judged-first escalate halt breaks that. No card lists the member whose doc holds that line, which is the `ParentAlign` type or its constructor near line 121.
- LOW — Card 1 (D1/D2), the loomcli package doc: `internal/loomcli/doc.go:17` describes pawl's Finalize as one that "lets a large merge go on to the gate in place of a reroute". That framed pawl as the exception to loom and darn. After D1 every recipe does it, and pawl's distinct flag `LargeMergeLands` is gone. No card lists `internal/loomcli/doc.go`.
- LOW — Card 1 (D1), the pawl recipe header: `contracts/recipes/pawl-recipe.yaml:31-34` explains pawl's lack of an `on_reroute` edge as "a conflict or a merge too large to land has no earlier row to return to". After D1, a conflict or large merge is no reroute cause for any recipe. Card 1 rewords the loom and darn recipe headers but not pawl's, which now gives a stale reason.
- LOW — Card 1 (D2), the test file header: `internal/landingshed/finalize_merge_test.go:3` says the file drives the merge "over a fake task merger, a fake resolver and the gate fixture". Card 1 deletes `fakeTaskMerger`, but the file header lies outside the `TestFinalize_MergeJudgement` span. `edit-test-file` bars listing the path, so Card 1's Intent should name the header rewording beside the `fakeTaskMerger` delete.

Calls made:
1. Grepped `contracts`, `docs`, `plugins`, `pattern` and `tools` for hand-back, overlap-note and large-merge wording, because the pre-read covered only the files `relevant.md` names.
2. Read `pawl-recipe.yaml:28-38` and grepped the specs for `hand_back`/`restep`, to check the pawl header and any other spec that lists the loop keys.
3. Ran `lym quarry resolve` on Card 4's and Card 2's listed glyphs (`statusCmd`, `runLoop`, `wireDarn`, `battenCLI.wire`, `ParentAlign.haltSpent`), because they were not quarried during exploration.
4. Grepped the tests for other closed key sets and for review-input, overlap and reroute wording, because the pre-read covered only the named tests.
5. Grepped non-test Go for large-merge, overlap-note and escalate-to-orch comments outside landingshed, because the pre-read did not cover them.
6. Read `internal/loomcli/doc.go:10-22`, because the previous grep hit line 17 and its context was missing.
