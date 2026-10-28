```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: what the diff's changes force outside the lines it touches**

Per change, the set I found:

1. **`acquireBranchLock` gained a `verb` parameter, the lock owner run id is now `"fabric-"+verb`, and `anchorLockOwnerRunID` was removed.**
   - Callers are `anchordeps.go:235` (passes "anchor") and `hubpull.go:30` (passes "pull"). Covered.
   - Nothing else in the tree reads `anchorLockOwnerRunID` or the string `fabric-anchor` as a run id. Grep hits for `.fabric-anchor` are the unrelated legacy marker file. Covered.
   - The anchor wait notice stays "lym fabric anchor: waiting…", so `anchor_test.go` is unaffected. Covered.
   - The release warn message changed to "a verb's parent lock is not released" with a `verb` attr. I found no assertion on the old text. Covered.

2. **A new `--hub` flag on `lym fabric pull`.**
   - The flag registration, the `RunE` branch, the help text and `fabriccli/doc.go` all carry it. Covered.
   - Nothing else asserts on `--hub` for this verb. The only other `--hub` hit is an unrelated reed watchdog flag. Covered.
   - `tools/sandbox/SANDBOX-FABRIC-SUITE.md` documents `lym fabric pull` scenarios and does not mention the hub pull. A sandbox suite is not a required doc here, so this is not a finding.

3. **New exported `Bolt.Pull`, `PullHub`, `PairLock` and the `HubPull*` types.**
   - `fabricengine/doc.go` has a new "The hub pull" section. Covered.
   - `docs/overview.md` has no `fabric pull` row, and no module row changed. Covered.
   - `PATTERN.md` and `pattern/` are untouched, which matches the record. No new cross-cutting invariant is added, so I raise nothing.

4. **The long help text of `lym fabric pull` was rewritten for existing, non-hub behaviour.**
   - The diff shortens the fast-forward and history-rewrite bullets in `weft_verbs.go:324-339`.
   - The decision record says "Out: changing what `lym fabric pull` without `--hub` does". The text change is documentation only, but it is an unrequested edit.
   - `fabricengine/doc.go:9` and `:13` still carry the original longer wording, so the two descriptions now differ in detail.
   - No test or fixture asserts the old help text; my grep found none.

Findings:

- **LOW, `scope`, `internal/fabriccli/weft_verbs.go:324-339`:** The diff trims the existing non-hub help bullets for the warp history rewrite (the "double-conflict" bullet text and the fast-forward bullet). The board entry and decision record do not ask for this. Evidence: the diff hunk at `weft_verbs.go:324-341`, plus the decision record's "Out: changing what `lym fabric pull` without `--hub` does". The fix is to restore the original bullets and keep only the appended `--hub` paragraph.

I found no other member of the affected set that the diff leaves wrong.

Tool calls (2):
1. A grep for `acquireBranchLock`, `anchorLockOwnerRunID`, `fabric-anchor` and the old warn text, for the old help phrases, and for `fabric pull` in the docs. The manifest lacked the full caller and reference set.
2. A grep for files mentioning `fabric pull`, and for other `--hub` flag uses. I needed this to find tests, docs and fixtures that assert pull behaviour.
