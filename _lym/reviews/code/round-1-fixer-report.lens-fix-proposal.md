```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1, F2, F3, F5–F8, F11, F12), looking for a case where the proposal is wrong or breaks something.

**Findings**

1. **LOW**, `pattern/PATTERN-run-fsm.md`, the added paragraph on `webster reset`.
   - **Summary:** The paragraph describes the reset's push order without saying it departs from the section's declared rule.
   - **Evidence:** The section's rule is "records side first, then the code branch", and the new text says the commit goes "after the reset moved the code branch on origin". So the reset pushes code first and records second.
   - **Effect:** A reader checking the rule against the paragraph sees a silent contradiction. The paragraph should call this an exception and say why it is safe. The "owed push holds the run" backstop it relies on is stated only by reference to the section below.

2. **LOW**, `internal/fabricengine/recordsgit.go`, `forceEmptyCommit := (len(snapshotTags) > 0 || len(trailers) > 0) && !unborn`.
   - **Summary:** A reset can now fail where it used to succeed as a no-op.
   - **Evidence:** The forced empty commit goes through `commitEmptySnapshot`, which calls `CommitEmpty`. `CommitEmpty` refuses with `gitrepo.ErrIndexNotEmpty` when anything unrelated is staged in the records index. `TestCommit_DirtyWeftIndex_UnchangedContentWithTags_SurfacesPartialCommitError` pins that refusal for tags.
   - **Effect:** A reset with a dirty records index now returns "the branch was reset but the fabric sync failed" after the code branch and origin have already moved. That is rare, and the trade is correct, but no test pins the trailer-and-dirty-index case.

3. **LOW**, `contracts/specs/loom-plan-checks-spec.md`, the added `overview-frame-edited` entry.
   - **Summary:** The way-forward sentence is ambiguous.
   - **Evidence:** "A change to its work is a follow-up card…" copies the `done-card-edited` phrasing, where "its work" means a card's work. The overview has no "work", and the review's wording was "carry the change in a follow-up card after the last begun batch".
   - **Effect:** Cosmetic. Rewording to "carry the change in a follow-up card" would remove the ambiguity.

**Challenged and held**

- **F3, `judgedKey`:** The same stop returning after an applied tackler resume keeps `judged` true, because its key is equal. A different stop reached by an arbiter resume gets `judged` false, which is the intended fix. A stop the tackler moved to and that then hands back by rule no longer sends the run-id line. The batten notice still goes out, so the parent hears about it either way.
- **F1 with the new test:** The `BatchStartWithNothingToClear` row fails when the forced empty commit is removed. The fixer ran this and got a failure.
- **F7:** The Card Index row replaces text that sits in the Card Index, since the frame hash stays equal and the test passes.
- **F8, F11, F12:** I found no breakage. `runID` is wired from a non-empty `WorktreeName`, so that assertion is not vacuous. The unborn-warp path returns before correspondence recording.

**Further tool calls:** none. My context held every file I needed.
