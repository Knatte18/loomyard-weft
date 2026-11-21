```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's three doc.go changes (and the summary.md edit) for consistency with their surroundings.

**Findings**

1. **MEDIUM — F1 step sentence contradicts the base steps.**
   - Changed text: `internal/adoptcli/doc.go`, "The steps run in order: start, each tier's start and end, then end."
   - The base doc's `# The steps` lists eight adopt steps: `hold`, `pairs`, `takeover`, `catchup`, `deploy`, `workspace`, `check`, `beam`. "Tier" appears nowhere in them (base lines 36-67).
   - "Start / each tier's start and end / end" is the reviewer's wording from a different engine, not this verb's.
   - `cli.go:47` says "The package doc holds the steps", and the new sentence names none of them.
   - The help text's claim therefore still doesn't hold, and the sentence is false for this package.
   - Fix: replace it with the real order, for example "hold, pairs, takeover, catchup, deploy, workspace, check, beam".
   - Mention the contracts a caller relies on: the check runs in each subject's worktree, `--no-orch` leaves each orch down, and a failed deploy or catch-up ends the adopt.

2. **MEDIUM — F2 overview in `internal/yamlengine/doc.go` is false.**
   - Changed text: "The file's values always win over the template's, and no operation replaces a value the file holds."
   - `SetValues` sets values over the file's, and `SetSparse` and `SetEntries` write new values.
   - `Merge` lays an upper layer over a lower one.
   - `FollowTemplate` replaces a unit left at the old template value.
   - Fix: restate it only for the operations it holds for (`Reconcile` and `FillMissing`), or drop the sentence.

3. **LOW — F2 per-name `Reconcile` bullet is garbled.**
   - Changed text: "The loaded value is unchanged by the names Reconcile drops, so a later template edit to that name reaches the file."
   - The base said the loaded value is unchanged, so a later template edit to that name reaches it.
   - "Reaches the file" is wrong: the dropped name is absent from the file, and the load fills it from the template.
   - `Reconcile`'s own doc in `reconcile.go:25-36` does not carry this rationale, so the "why" is worth keeping, but it should read "reaches the loaded value".

4. **LOW — F3 `internal/batcher/doc.go` leaves "start base" undefined.**
   - Changed text: "A fork inherits the orchestrating session's context, which is the start base plus orientation…".
   - The cut removed the `StartBase` definition, so the term now has no definition in the file.
   - `orientation` and `batch_growth` are only defined in a later paragraph.
   - The cut also removed the sentences that gave "Webster computes it from the loaded text…".
   - The `PeakContext` doc at `estimate.go:207-209` uses "base" too, so the term is probably documented on `StartBase` itself.
   - Cross-check that doc, or add half a sentence ("what the orchestrating session loads at start").

5. **NIT — F4 `_lym/landing/summary.md` is edited but not committed.** The fixer must not run git on `_lym`, so this is expected. The report should say the edit is a working-tree change only. The "130 of the 161 files" edit is consistent with the Size table row.

No other twin was left behind: `cli.go:47` still names the steps, the hold and the envelope, and the envelope fields now match the base list.

**Tool calls**
- `git show 3d3b4812:internal/adoptcli/doc.go | sed -n 36,75p`: read the base step list, which my context only had as elided slices.
- `grep -n -i 'step\|tier' internal/adoptcli/cli.go`: confirmed the help-text claim at `cli.go:47` and whether "tier" is used there.
