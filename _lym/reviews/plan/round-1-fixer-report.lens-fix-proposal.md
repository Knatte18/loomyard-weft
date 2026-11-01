```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: challenging the review proposals the fixer carried out as written.

Findings:

1. LOW. Card 19 (old 18), F1's added Create lines `plan:internal/heddlecli#RunCLI` and `#RunCLIIn`, plus the RunCLI Intent sentence.
   - Summary: the proposal adds the two seam functions but leaves out the compile-time pin that every other module's seam functions sit in, so heddlecli's signatures stay unpinned.
   - Evidence: `cmd/lym/seamsignature_test.go:1-2` says it "pins every module's RunCLI(io.Writer, []string) int seam function, and every module's RunCLIIn … to their exact signatures at compile time". It lists each module (`boardcli.RunCLIIn`, `burlercli.RunCLIIn`, … lines 61-68). Card 19 edits only `cmd/lym#newRoot`.
   - I found no completeness guard that would force the entry, so nothing fails. The sibling pin is still stale.

2. LOW. Card 11 (old 10), F19's cut of four `TestTable` rows and the sentence "covered by `boardengine`'s `TestFabricReplay`, which card 12 runs over this merge unchanged".
   - Summary: from card 11's own gate until card 12 lands, nothing tests `Table.Change`'s applied path, its both-sides-lost path or `Same`'s false case. Card 17's heddle replayer also leans on `entrymerge` and sits two packages away from `TestFabricReplay`.
   - Case: a `Change` that returns `origin` unchanged for every field passes every kept `TestTable` row at card 11. It fails only at card 12, which this card's gate never runs.
   - The proposal moves the coverage of the package's core contract one card later and into another package.

3. LOW. Card 10 (old 9), F8's new Where-it-fires bullet for `Fabric.FastForwardPair` and its `TestWorktreeSeam_Events` row.
   - Summary: D6 says "a landing that brings a new `AGENTS.md` … renders nothing". `FastForwardPair`'s only callers are the landing's Finalize (`landingshed/finalize.go:379`, `finalize_pr.go:247`), yet the card fires "took in" there.
   - The card's reading is defensible: D6's exception names bringing commits into a beam, and this moves the parent pair to origin. But the bullet does not say why the landing exception does not apply, so the card can be read as contradicting D6.
   - I found no lock held around the call at `finalize.go:360-379`, so the render's Bolt write lock cannot self-block.

4. No finding on the other proposals carried out as written: F2, F3, F4, F10, F11, F13, F14, F15, F16, F18, F20 and F7's card.
   - F13's row holds: `RequireWarpWorktree` (`fabric.go:139,143`) has distinct texts for the records sibling and the Bolt checkout.
   - F14 holds: no test pins "failed board replay".
   - F2 holds: `AssertImportAllowlist` checks direct imports of `internal/runproc` only (`scankit.go:342-351`).

Tool calls:
- `landingshed/finalize.go:360-382`, `restoreReplayTip`'s message in tests, and the cmd/lym RunCLIIn guards plus `scankit.AssertImportAllowlist`: to check F8's lock context, F14's pinned text and F1/F2's sibling guards, none of which my context held.
- `seamsignature_test.go` contents and `RequireWarpWorktree`'s refusal texts: to see whether the seam pin is per-module and what F13's row would assert.
- A grep for a guard that every cli module is listed in `seamsignature_test.go`: to judge whether finding 1 fails a test or only leaves the pin stale.
